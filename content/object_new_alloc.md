# 对象的创建与内存分配

你知道 ```ObjectA obj = new ObjectA();``` 背后的都做了哪些工作吗？

![image](https://user-images.githubusercontent.com/13992911/116414090-c3ce2980-a86a-11eb-85a1-35d443a4d411.png)

## 对象分配过程
1. 依据逃逸分析，判断是否能栈上分配？
如果可以，使用标量替换方式，把对象分配到VM Stack中。如果 线程销毁或方法调用结束后，自动销毁，不需要 GC 回收器 介入。
否则，继续下一步。
2. 判断是否大对象？
如果是，直接分配到堆上 Old Generation 老年代上。如果对象变为垃圾后，由老年代GC 收集器（比如 Parallel Old, CMS, G1）回收。
否则，继续下一步。
3. 判断是否可以在 TLAB中分配？
如果是，在 TLAB中分配堆上Eden区。否则，在 TLAB外堆上的Eden区分配。

## 栈上分配
本质上是JVM提供的一个优化技术。

基本思想：将线程私有的对象打散分配在栈 VM Stack上
优点：
可以在函数调用结束后自行销毁对象，不需要垃圾回收器的介入，有效避免垃圾回收带来的负面影响
栈上分配速度快，提高系统性能

局限性： 
栈空间小，对于大对象无法实现栈上分配

技术基础： **逃逸分析**、**标量替换**

## TLAB 分配
TLAB，全称Thread Local Allocation Buffer, 即：线程本地分配缓存。这是一块线程专用的内存分配区域。
TLAB占用的是eden区的空间。
在TLAB启用的情况下（默认开启），JVM会为每一个线程分配一块TLAB区域。

### 为什么需要TLAB？
这是为了加速对象的分配。
由于对象一般分配在堆上，而堆是线程共用的，因此可能会有多个线程在堆上申请空间，而每一次的对象分配都必须线程同步，会使分配的效率下降。
考虑到对象分配几乎是Java中最常用的操作，因此JVM使用了TLAB这样的线程专有区域来避免多线程冲突，提高对象分配的效率。

局限性： TLAB空间一般不会太大（占用eden区），所以大对象无法进行TLAB分配，只能直接分配到堆 Heap上。

## 何为大对象
大对象的 JVM 参数如下：

大对象到底多大：```-XX:PreTenureSizeThreshold=n```
（仅适用于 DefNew / ParNew新生代垃圾回收器 ） bugs.openjdk.java.net/browse/JDK-…
G1回收器的大对象判断，则依据Region的大小（```-XX:G1HeapRegionSize```）来判断，如果对象大于Region 50%以上，就判断为大对象Humongous Object。

