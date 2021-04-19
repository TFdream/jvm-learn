# JVM垃圾回收

## JVM常用基础参数
* -Xms 初始大小内存，默认为物理内存的1/64 等价于-XX:InitialHeapSize
* -Xmx 最大分配内存，默认为物理内存的1/4 等价于-XX:MaxHeapSize
* -Xss 设置单个线程栈的大小，默认512K～1024K 等价于-XX:ThreadStackSize
* -Xmn 设置年轻代大小
* -XX:PrintGCDetails 输出详细GC收集日志信息
* -XX:SurvivorRatio 设置新生代中eden和S0/S1空间的比例 默认-XX:SurvivorRatio=8
* -XX:NewRatio 配置年轻代与老年代在堆结构中的占比 默认-XX:NewRatio=2（即年轻代为老年代的1/2）
* -XX:MaxTenuringThreshold 设置垃圾在年轻代的最大年龄，默认为15


## jdk各版本默认垃圾收集器

在JDK 9中，G1被提议设置为默认垃圾收集器（JEP 248）。


## G1算法
[Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)
