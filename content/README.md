# JVM垃圾回收

## 垃圾回收器总览
![image](https://user-images.githubusercontent.com/13992911/115195314-329bdc00-a121-11eb-87d6-a6c98712c0b1.png)

新生代可配置的回收器：Serial、ParNew、Parallel Scavenge

老年代配置的回收器：CMS、Serial Old、Parallel Old

新生代和老年代区域的回收器之间进行连线，说明他们之间可以搭配使用。

## minor GC和Full GC的区别是什么？触发条件分别是什么？
[minor GC和Full GC的区别是什么？触发条件分别是什么？](https://github.com/TFdream/jvm-learning/blob/main/content/ygc_full_GC.md)

## jdk各版本默认垃圾收集器
### 如何查看jdk默认垃圾收集器？
> -XX:+PrintCommandLineFlags 参数可查看默认设置收集器类型

> -XX:+PrintGCDetails 亦可通过打印的GC日志的新生代、老年代名称判断

### JDK 1.7
jdk1.7 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代），参考[知乎这篇文章](https://www.zhihu.com/question/56344485)，一楼有R大的回答:

> 这个问题的答案取决于JDK版本，在2012年默认值改变过一次。自JDK7u4开始的JDK7u系列与JDK8系列，如果指定了：-XX:+UseParallelGC，则会默认开启：```XX:+UseParallelOldGC```。
> 在这个改变之前，即便选择了ParallelGC，默认情况下ParallelOldGC并不会随即开启，而是要自己通过 -XX:+UseParallelOldGC 去选定。

### JDK 8
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

### JDK 9
从JDK 9开始G1替代并行垃圾回收器成为JVM中默认的垃圾回收器（具体可见JEP提案248，链接：https://openjdk.java.net/jeps/248），并且官方将CMS标记为丢弃（具体可见JEP提案291，链接：https://openjdk.java.net/jeps/291）。G1能够脱颖而出，成为最大的赢家，其最主要的原因就是在过去几年间，众多使用者使用G1之后发现G1的性能表现的非常优秀。

## JVM常用基础参数
| 参数 | 说明|
| --- | --- |
| -Xms | 初始大小内存，默认为物理内存的1/64 等价于-XX:InitialHeapSize|
| -Xmx | 最大分配内存，默认为物理内存的1/4 等价于-XX:MaxHeapSize | 
| -Xss | 设置单个线程栈的大小，默认512K～1024K 等价于-XX:ThreadStackSize | 
| -Xmn | 设置年轻代大小 | 
| -XX:NewRatio | 配置年轻代与老年代在堆结构中的占比 默认-XX:NewRatio=2（即年轻代为老年代的1/2） | 
| -XX:SurvivorRatio | 设置新生代中eden和S0/S1空间的比例 默认-XX:SurvivorRatio=8 | 
| -XX:MetaspaceSize | 设置元空间大小。元空间本质跟永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的区别在于：元空间并不在虚拟机中，而是使用本机内存。因此，元空间大小仅受本地内存限制。 | 
| -XX:MaxTenuringThreshold | 设置垃圾在年轻代的最大年龄，默认为15 |
| -XX:PrintGCDetails | 输出详细GC收集日志信息 | 

## JVM高频面试题
* [JVM高频面试题](https://github.com/TFdream/jvm-learning/blob/main/content/jvm_interview_question.md)

## G1算法
[Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)
