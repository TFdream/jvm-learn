# Java对象内存布局
> 注：本篇文章内容主要参考自周志明 - 《深入理解Java虚拟机（第二版）》

本文只包含简单java对象的内存布局，不考虑继承的情况。

Java对象的内存布局：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。无论是32位还是64位的HotSpot，使用的都是8字节对齐。 也就是说每个java对象，占用的字节数都是8的整数倍。
**静态数据成员，方法成员为类的所有实例共享，不保存在某个对象实例里**。

当然，java作为一种面向对象的语言，更多的情况需要考虑对象的内存布局，java对于对象所占内存大小需要分两种情况考虑：
| 对象类型  | 内存布局构成 |
| 一般非数组对象 | 8个字节对象头(mark) + 4/8字节对象指针 + 数据区 + padding内存对齐(按照8的倍数对齐) |
| 数组对象 | 8个字节对象头(mark) + 4/8字节对象指针 + 4字节数组长度 + 数据区 + padding内存对齐(按照8的倍数对齐) |

可以看到数组类型对象和普通对象的区别仅在于4字节数组长度的存储区间。而对象指针究竟是4字节还是8字节要看是否开启指针压缩。Oracle JDK从6 update 23开始在64位系统上会默认开启压缩指针。启用指针压缩使用: -XX:+UseCompressedOops

## 对象内存布局
* 对象头：存储对象自身的运行时数据，包括：
   * Mark Word（在32bit和64bit虚拟机上长度分别为32bit和64bit），包含如下信息：
     * 对象hashCode
     * 对象GC分代年龄
     * 锁状态标志（轻量级锁、重量级锁）
     * 线程持有的锁（轻量级锁、重量级锁）
     * 偏向锁相关：偏向锁、自旋锁、轻量级锁以及其他的一些锁优化策略是JDK1.6加入的，这些优化使得Synchronized的性能与ReentrantLock的性能持平，在Synchronized可以满足要求的情况下，优先使用Synchronized
   * 类型指针：对象指向类元数据的指针（32bit-->32bit，64bit-->64bit(未开启压缩指针)，32bit(开启压缩指针)）
     * JVM通过这个指针来确定这个对象是哪个类的实例（根据对象确定其Class的指针）。
* 实例数据：对象真正存储的有效信息
* 对齐填充：JVM要求对象的大小必须是8的整数倍，若不是，需要补位对齐。

## 如何计算对象占用内存大小
如何手动计算对象大小的文章，总结了几点：
1. 基本数据类型占用的字节数，JVM规范中有明确的规定，无论是在32位还是64位的虚拟机，占用的内存大小是相同的。
2. reference类型在32位JVM下占用4个字节，但是在64位下可能占用4个字节或8个字节，这取决于是否启用了64位JVM的指针压缩参数UseCompressedOops。
3. new Object()这个对象在32位JVM上占8个字节，在64位JVM上占16个字节。
4. 开启(-XX:+UseCompressedOops)指针压缩，对象头占12字节; 关闭(-XX:-UseCompressedOops)指针压缩,对象头占16字节。
5.64位JVM上，数组对象的对象头占用24个字节，启用压缩之后占用16个字节。之所以比普通对象占用内存多是因为需要额外的空间存储数组的长度。

笔者了解到的有3种方式：
* 通过Instrumentation的getObjectSize方法来获取；
* 通过Unsafe的objectFieldOffset方法可以间接推断出对象大小；
* [openjdk jol](http://openjdk.java.net/projects/code-tools/jol/)提供了 ClassLayout能够输出输出对象占用大小信息；

## 示例
假设对象 ObjectA 如下：
```

    private static class ObjectA {
        String str;   // 4
        int i1;       // 4
        byte b1;      // 1
        byte b2;      // 1
        int i2;       // 4
        ObjectB obj;  //4
        byte b3;      // 1
    }

    private static class ObjectB {
    }
```

### 1、通过Instrumentation
这种方法得到的是Shallow Size，即遇到引用时，只计算引用的长度，不计算所引用的对象的实际大小。如果要计算所引用对象的实际大小，必须通过递归的方式去计算。
```
import java.lang.instrument.Instrumentation;

public class ObjectShallowSize {
    private static Instrumentation inst;  
    public static void premain(String agentArgs, Instrumentation instP){
        inst = instP; 
    }
        
    public static long sizeOf(Object obj){
        return inst.getObjectSize(obj);
    }
 }
```
### 2、通过Unsafe

```
import sun.misc.Unsafe;
import java.lang.reflect.Field;

/**
 * @author Ricky Fung
 */
public class ObjectSize {

    private final static Unsafe UNSAFE;

    // 只能通过反射获取Unsafe对象的实例
    static {
        try {
            UNSAFE = (Unsafe) Unsafe.class.getDeclaredField("theUnsafe").get(null);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public void sizeOf(Object obj) {
        Field[] fields = obj.getClass().getDeclaredFields();
        for (Field field : fields) {
            System.out.println(field.getName() + "---offSet:" + UNSAFE.objectFieldOffset(field));
        }
    }
}
```
输出结果如下：
```
str---offSet:24
i1---offSet:12
b1---offSet:20
b2---offSet:21
i2---offSet:16
obj---offSet:28
b3---offSet:22
```

我们同样可以算得对象实际占用的内存大小：
> Size(ObjectA) = Size(对象头(_mark)) + size(oop指针) + size(排序后数据区)  =  8 + 4 + (28+4-12)  =  32.

我们再回过头来，看我们在通过代码获取对象所占内存大小之前的预估值40。比我们实际算出来的值多了8个字节。通过Unsafe打印的详细信息，我们不难想到这其实是由hotspot创建对象时的排序决定的：

HotSpot创建的对象的字段会先按照给定顺序排列，默认的顺序为：从长到短排列，引用排最后: long/double –> int/float –> short/char –> byte/boolean –> Reference。

所以我们重新计算对象所占内存大小得：

> Size(ObjectA) = Size(对象头(_mark)) + size(oop指针) + size(排序后数据区)
> Size(ObjectA) = 8 + 4 + 4(int) + 4(int) + byte(1) + byte(1) + 2(padding) + 4(String) + 4(ObjectB指针)
> Size(ObjectA) = 32

与上面计算结果一致。
### 3、jol
首先，添加jol依赖：
```
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.15</version>
    <scope>provided</scope>
</dependency>
```

接下来就使用 ClassLayout的 parseClass或者 parseInstance来查看内存布局了，如下：
```

    public static void main(String[] args) {
        ObjectA objectA = new ObjectA();

        //System.out.println(ClassLayout.parseClass(ObjectA.class).toPrintable());
        //System.out.println("==========");
        
        System.out.println(ClassLayout.parseInstance(objectA).toPrintable());
    }
```

运行这段代码，可以看到jol输出如下信息：
```
com.renzhenmall.oms.ObjectSize$ObjectA object internals:
OFF  SZ                                     TYPE DESCRIPTION               VALUE
  0   8                                          (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4                                          (object header: class)    0xf800c144
 12   4                                      int ObjectA.i1                0
 16   4                                      int ObjectA.i2                0
 20   1                                     byte ObjectA.b1                0
 21   1                                     byte ObjectA.b2                0
 22   1                                     byte ObjectA.b3                0
 23   1                                          (alignment/padding gap)   
 24   4                         java.lang.String ObjectA.str               null
 28   4   com.renzhenmall.oms.ObjectSize.ObjectB ObjectA.obj               null
Instance size: 32 bytes
Space losses: 1 bytes internal + 0 bytes external = 1 bytes total

```

可以看到jol工具，能够显示出对象头的大小，以及每个实例字段的偏移，进而计算对象占用的内存大小。

jol工具还提供了一个命令行工具```jol-cli```，包含了main方法能够直接在命令行运行，查看类的布局信息。


