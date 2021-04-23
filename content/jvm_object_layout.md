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
* 通过Instrumentation来获取；
* 通过Unsafe的使用，后面我会专门开一个专题来详细讲述，这里暂时让我们来见识下Unsafe的神奇之处。
* [openjdk jol](http://openjdk.java.net/projects/code-tools/jol/)提供了 ClassLayout能够输出输出对象占用大小信息；

### 通过Instrumentation
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
### 通过Unsafe

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

### openjdk jol

