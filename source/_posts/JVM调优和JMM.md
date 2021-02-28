---
title: JVM 调优和JMM
date: 2021-2-28
cover:
top_img:
categories: JVM
tags: 
mathjax: true
katex: true
---
# jvm 
https://www.bilibili.com/video/BV1ai4y1g7vv?from=search&seid=6275678583243369276

## 一、 jvm 调优

要减少 stop the world 的次数, 优先减少full gc次数

1 一个调优思想，让对象在年轻代就消亡，尽量少进入老年代，这样就可以减少full gc


## 二、 jmm 内存模型和JSR133

java内存模型可以说是java并发内存模型，

### 1、可见性

工作内存和主内存，volatile, 将变量共，缓存锁定机制。

早期volatile 是总线锁，在该变量的主内存加锁。

现在缓存一致性协议（MESI）

总线嗅探机制
E ：独占状态
S ：共享状态
M ：加锁状态
I ：失效状态

happen before Aqs NACOS springcloud alibaba sentinel


### 2、原子性

原子性指的是一个操作是不可中断的，即使是在多线程环境下，一个操作一旦开始就不会被其他线程影响。比如a=0，作为一个写操作就是原子操作。

那么JMM中是如何保证其它操作的原子性呢？那就是synchronized关键字和Lock接口，因为synchronized和Lock能够保证任一时刻只有一个线程访问该代码块。

### 3、有序性

为了提高程序的执行效率，在不影响程序的运行结果的前提下（仅针对单线程），指令的执行顺序可以被改变，这就是指令重排序。

有序性主要就是在多线程的场景下，保证不进行指令重排，使指令的执行顺序与程序中的代码的顺序一致。

在JMM中，可以通过volatile关键字来保证一定的“有序性”，因为它提供了内存屏障防止指令重排。当然synchronized和Lock也可以保证有序性，因为它们另代码变成了同步代码块，自然不会有指令乱序问题。

## 三、 JVM CPU调度
JVM创建线程栈时，JVM会创建KLT调用操作系统生成一个线程。
hsdis 查看汇编的软件

## 四、java程序的执行流程

- java程序首先经过编译器javac的编译，然后交给JVM
- 字节码的运行过程：
    + 通过类装载子系统来加载数据到运行时数据区
    + 通过字节码执行引擎来执行代码

