# JVM垃圾回收

## 垃圾回收器总览
![image](https://user-images.githubusercontent.com/13992911/115195314-329bdc00-a121-11eb-87d6-a6c98712c0b1.png)

新生代可配置的回收器：Serial、ParNew、Parallel Scavenge

老年代配置的回收器：CMS、Serial Old、Parallel Old

新生代和老年代区域的回收器之间进行连线，说明他们之间可以搭配使用。


## jdk各版本默认垃圾收集器
JDK8默认使用的垃圾收集器可通过如下命令查看：
```
RickydeMacBook-Pro:mall-admin-service apple$ java -XX:+PrintCommandLineFlags -version
```
结果如下：
```
-XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

```-XX:+UseParallelGC``` 即 Parallel Scavenge + Parallel Old。

在JDK 9中，G1被提议设置为默认垃圾收集器（JEP 248）。

## JVM常用基础参数
| 参数 | 说明|
| -Xms | 初始大小内存，默认为物理内存的1/64 等价于-XX:InitialHeapSize|
| -Xmx | 最大分配内存，默认为物理内存的1/4 等价于-XX:MaxHeapSize | 
| -Xss | 设置单个线程栈的大小，默认512K～1024K 等价于-XX:ThreadStackSize | 
| -Xmn | 设置年轻代大小 | 
| -XX:MetaspaceSize | 设置元空间大小。元空间本质跟永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的区别在于：元空间并不在虚拟机中，而是使用本机内存。因此，元空间大小仅受本地内存限制。 | 
| -XX:SurvivorRatio | 设置新生代中eden和S0/S1空间的比例 默认-XX:SurvivorRatio=8 | 
| -XX:NewRatio | 配置年轻代与老年代在堆结构中的占比 默认-XX:NewRatio=2（即年轻代为老年代的1/2） | 
| -XX:MaxTenuringThreshold | 设置垃圾在年轻代的最大年龄，默认为15 |
| -XX:PrintGCDetails | 输出详细GC收集日志信息 | 

## G1算法
[Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)
