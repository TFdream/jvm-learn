# 对象的创建与内存分配

你知道 ```ObjectA obj = new ObjectA();``` 背后的都做了哪些工作吗？

![image](https://user-images.githubusercontent.com/13992911/116414090-c3ce2980-a86a-11eb-85a1-35d443a4d411.png)

## 对象分配过程
1.依据逃逸分析，判断是否能栈上分配？
如果可以，使用标量替换方式，把对象分配到VM Stack中。如果 线程销毁或方法调用结束后，自动销毁，不需要 GC 回收器 介入。
否则，继续下一步。
2. 判断是否大对象？
如果是，直接分配到堆上 Old Generation 老年代上。如果对象变为垃圾后，由老年代GC 收集器（比如 Parallel Old, CMS, G1）回收。
否则，继续下一步。
3. 判断是否可以在 TLAB中分配？
如果是，在 TLAB中分配堆上Eden区。否则，在 TLAB外堆上的Eden区分配。
