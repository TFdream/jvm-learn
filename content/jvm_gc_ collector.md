# 7种jvm垃圾回收器，这次全部搞懂

之前我们讲解了jvm的组成结构与垃圾回收算法等知识点，今天我们来讲讲jvm最重要的堆内存是如何使用垃圾回收器进行垃圾回收，并且如何使用命令去配置使用这些垃圾回收器。


## 堆内存垃圾回收过程
### 第一步
新生成的对象首先放到Eden区，当Eden区满了会触发Minor GC。

### 第二步
第一步GC活下来的对象，会被移动到survivor区中的S0区，S0区满了之后会触发Minor GC，S0区存活下来的对象会被移动到S1区，S0区空闲。

S1满了之后在GC，存活下来的再次移动到S0区，S1区空闲，这样反反复复GC，每GC一次，对象的年龄就涨一岁，达到某个值后（15），就会进入老年代。

### 第三步
在发生一次Minor GC后（前提条件），老年代可能会出现Major GC，这个视垃圾回收器而定。
* Full GC触发条件
* 手动调用System.gc，会不断的执行Full GC
* 老年代空间不足/满了
* 方法区空间不足/满了

## 回收哪些区域的对象
需要注意的是，JVM GC只回收堆内存和方法区内的对象。而栈内存的数据，在超出作用域后会被JVM自动释放掉，所以其不在JVM GC的管理范围内。

## 垃圾回收器总览
![image](https://user-images.githubusercontent.com/13992911/115179669-6ddce180-a106-11eb-9df9-4e932c371256.png)

新生代可配置的回收器：Serial、ParNew、Parallel Scavenge

老年代配置的回收器：CMS、Serial Old、Parallel Old

新生代和老年代区域的回收器之间进行连线，说明他们之间可以搭配使用。

## 新生代垃圾回收器
### 1、Serial 垃圾回收器
Serial收集器是最基本的、发展历史最悠久的收集器。俗称为：串行回收器，采用复制算法进行垃圾回收

特点
串行回收器是指使用单线程进行垃圾回收的回收器。每次回收时，串行回收器只有一个工作线程。

对于并行能力较弱的单CPU计算机来说，串行回收器的专注性和独占性往往有更好的性能表现。

它存在Stop The World问题，及垃圾回收时，要停止程序的运行。

使用-XX:+UseSerialGC参数可以设置新生代使用这个串行回收器


### 2、ParNew 垃圾回收器
ParNew其实就是Serial的多线程版本，除了使用多线程之外，其余参数和Serial一模一样。俗称：并行垃圾回收器，采用复制算法进行垃圾回收

特点
ParNew默认开启的线程数与CPU数量相同，在CPU核数很多的机器上，可以通过参数-XX:ParallelGCThreads来设置线程数。

**它是目前新生代首选的垃圾回收器，因为除了ParNew之外，它是唯一一个能与老年代CMS配合工作的。**

它同样存在Stop The World问题

使用-XX:+UseParNewGC参数可以设置新生代使用这个并行回收器

### 3、ParallelGC 回收器
ParallelGC使用复制算法回收垃圾，也是多线程的。

**特点**
就是非常关注系统的吞吐量，吞吐量=代码运行时间/(代码运行时间+垃圾收集时间)，系统吨吐量优先。

-XX:MaxGCPauseMillis：设置最大垃圾收集停顿时间，可用把虚拟机在GC停顿的时间控制在MaxGCPauseMillis范围内，如果希望减少GC停顿时间可以将MaxGCPauseMillis设置的很小，但是会导致GC频繁，从而增加了GC的总时间，降低了吞吐量。所以需要根据实际情况设置该值。

-Xx:GCTimeRatio：设置吞吐量大小，它是一个0到100之间的整数，默认情况下他的取值是99，那么系统将花费不超过1/(1+n)的时间用于垃圾回收，也就是1/(1+99)=1%的时间。

另外还可以指定-XX:+UseAdaptiveSizePolicy打开自适应模式，在这种模式下，新生代的大小、eden、from/to的比例，以及晋升老年代的对象年龄参数会被自动调整，以达到在堆大小、吞吐量和停顿时间之间的平衡点。

使用-XX:+UseParallelGC参数可以设置新生代使用这个并行回收器

## 老年代垃圾回收器
### 1、SerialOld 垃圾回收器
SerialOld是Serial回收器的老年代回收器版本，它同样是一个单线程回收器。

使用算法：标记 - 整理算法
用途
* 一个是在JDK1.5及之前的版本中与Parallel Scavenge收集器搭配使用，
* 另一个就是作为CMS收集器的后备预案，如果CMS出现Concurrent Mode Failure，则SerialOld将作为后备收集器。

### 2、ParallelOldGC 回收器
老年代ParallelOldGC回收器也是一种多线程的回收器，和新生代的ParallelGC回收器一样，也是一种关注吞吐量的回收器，他使用了**标记压缩**算法进行实现。

-XX:+UseParallelOldGc进行设置老年代使用该回收器

-XX:+ParallelGCThreads也可以设置垃圾收集时的线程数量。

### 3、CMS 回收器
CMS全称为:Concurrent Mark Sweep意为并发标记清除，他使用的是**标记清除法**。主要关注**系统停顿时间**。

使用-XX:+UseConcMarkSweepGC进行设置老年代使用该回收器。

使用-XX:ConcGCThreads设置并发线程数量。

特点
CMS并不是独占的回收器，也就说CMS回收的过程中，应用程序仍然在不停的工作，又会有新的垃圾不断的产生，所以在使用CMS的过程中应该确保应用程序的内存足够可用。

CMS不会等到应用程序饱和的时候才去回收垃圾，而是在某一阀值的时候开始回收，回收阀值可用指定的参数进行配置：-XX:CMSInitiatingoccupancyFraction来指定，默认为68，也就是说当老年代的空间使用率达到68%的时候，会执行CMS回收。

如果内存使用率增长的很快，在CMS执行的过程中，已经出现了内存不足的情况，此时CMS回收就会失败，虚拟机将启动老年代串行回收器；SerialOldGC进行垃圾回收，这会导致应用程序中断，直到垃圾回收完成后才会正常工作。

这个过程GC的停顿时间可能较长，所以-XX:CMSInitiatingoccupancyFraction的设置要根据实际的情况。

之前我们在学习算法的时候说过，标记清除法有个缺点就是存在内存碎片的问题，那么CMS有个参数设置-XX:+UseCMSCompactAtFullCollecion可以使CMS回收完成之后进行一次碎片整理。

-XX:CMSFullGCsBeforeCompaction参数可以设置进行多少次CMS回收之后，对内存进行一次压缩。

### G1 回收器
