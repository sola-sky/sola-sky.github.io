---
title: 单例模式之DCL
date: 2016-05-25 20:31:12
tags:
---

### volatile

1.给SingletonExample的实例分配内存。

2.初始化SingletonExample的构造器

3.将instance对象指向分配的内存空间（注意到这步instance就非null了）。

但是，由于Java编译器允许处理器乱序执行（out-of-order），以及JDK1.5之前JMM（Java Memory Medel）中Cache、寄存器到主内存回写顺序的规定，上面的第二点和第三点的顺序是无法保证的，也就是说，执行顺序可能是1-2-3也可能是1-3-2，如果是后者，并且在3执行完毕、2未执行之前，被切换到线程二上，这时候instance因为已经在线程一内执行过了第三点，instance已经是非空了，所以线程二直接拿走instance，然后使用，然后顺理成章地报错，而且这种难以跟踪难以重现的错误估计调试上一星期都未必能找得出来，真是一茶几的杯具啊。

#### 来源

1. [浅谈设计模式](http://http://www.cnblogs.com/techyc/p/3529983.html)
2. [Java 单例真的写对了么?](http://http://www.race604.com/java-double-checked-singleton)

