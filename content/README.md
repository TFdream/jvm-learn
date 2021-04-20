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
JDK8默认使用 ParallelGC 垃圾收集器，即 Parallel Scavenge + Parallel Old 组合。

### JDK 9
从JDK 9开始G1替代并行垃圾回收器成为JVM中默认的垃圾回收器（具体可见[JEP提案248](https://openjdk.java.net/jeps/248)），并且官方将CMS标记为丢弃（具体可见[JEP提案291](https://openjdk.java.net/jeps/291)）。G1能够脱颖而出，成为最大的赢家，其最主要的原因就是在过去几年间，众多使用者使用G1之后发现G1的性能表现的非常优秀。

## JVM常用基础参数
[JVM常用配置参数](https://github.com/TFdream/jvm-learning/issues/11)

## JVM高频面试题
* [JVM高频面试题](https://github.com/TFdream/jvm-learning/blob/main/content/jvm_interview_question.md)

## G1算法
[Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)
