---
title: 指令重排和happen-before
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 7 指令重排与happen-before
指令重排可以在CPU闲置（等待变量装载，对应CPU中的不同硬件。比如某一硬件执行完指令A中的命令，空闲后立刻执行B中命令，减少等待时间[指令重排意义](https://blog.csdn.net/u013099854/article/details/105664575/)）时*，先执行其他指令，提高性能。分为：
- 编译器优化重排：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令并行重排：现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性（即后一个执行的语句无需依赖前面执行的语句的结果），处理器可以改变语句对应的机器指令的执行顺序。
- 内存系统重排*：由于处理器使用缓存和读写缓存冲区，这使得加载（load）和存储（store）操作看上去可能是在乱序执行，因为三级缓存的存在，导致内存与缓存的数据同步存在时间差。

指令重排在单线程时可以保证串行语义一致，但多线程时可能出现问题。

JMM能保证指令重排后，不改变程序的执行结果：
- 单线程肯定能保证
- 多线程需要正确同步后，可以保证

## 7.1 happens-before来保证指令的运行结果

happens-before规范用来保证两个操作之间的执行顺序。这两个操作可以是一个线程内的，也可以是多线程的。

- 如果A happens-before B, 则A在内存上做的操作对B都是可见的。

Java中天然的happens-before关系：
- 程序顺序规则：一个线程中的每一个操作，happens-before与该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before与任意后续对这个volatile域的读。
- 传递性：如果A happens-before B， 且B happens-before C， 那么A happens-before C。
- start规则：如果线程A执行操作`ThreadB.start()`启动线程B，那么A线程的`ThreadB.start()`操作happen-before于线程B中的任意操作
- join规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

## 7.2 与final相关的重排序规则

1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作不能重排序。
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

- 写final域的重排序规则可以确保：在引用变量为任意线程可见之前，该引用变量指向的对象的final域已经在构造函数中被正确初始化过了。
- 还需要保证：在构造函数内部，不能让这个被构造对象的引用为其他线程可见，也就是对象对象引用不能再构造函数中“逸出”。

