## GC分类
针对HotSpot VM的实现，它里面的GC其实准确分类只有两大种：
* Partial GC：并不收集整个GC堆的模式
   * Young GC：只收集young gen的GC
   * Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
   * Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
* Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

**Major GC通常是跟full GC是等价的，收集整个GC堆**。但因为HotSpot VM发展了这么多年，外界对各种名词的解读已经完全混乱了，当有人说“major GC”的时候一定要问清楚他想要指的是上面的full GC还是old GC。

Full GC定义是相对明确的，就是针对整个新生代、老生代、元空间（metaspace，java8以上版本取代perm gen）的全局范围的GC；

## Young GC的触发条件
当young gen中的eden区分配满的时候触发。注意young GC中有部分存活对象会晋升到old gen，所以young GC后old gen的占用量通常会有所升高。

## Full GC的触发条件
Full GC的触发条件如下：
* full GC：当准备要触发一次young GC时，如果发现统计数据说之前young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC（因为HotSpot VM的GC里，除了CMS的concurrent collection之外，其它能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先触发一次单独的young GC）；
* 如果有perm gen的话，要在perm gen分配空间但已经没有足够空间时，也要触发一次full GC；
* 或者System.gc()、heap dump带GC，默认也是触发full GC。


## 相关资料
* [RednaxelaFX Major GC和Full GC的区别是什么？触发条件呢？](https://www.zhihu.com/question/41922036)
