# JVM内存分布
java内存区域主要分程序计数器、Java虚拟机栈、本地方法栈、Java堆、方法区、直接内存。其中程序计数器、Java虚拟机栈、本地方法栈属于线程隔离，即他们都有自己的线程归属，其他属于线程共享的。

![image](https://user-images.githubusercontent.com/13992911/121171986-c79d9480-c889-11eb-9c08-4380079ac6f3.png)
