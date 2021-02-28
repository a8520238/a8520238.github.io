---
title: java内存模型
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 6 Java内存模型

## 6.1 并发模型

线程间进行通信和同步，可以通过2种方式的并发模型来实现：
1. 消息传递并发模型
2. 共享内存并发模型

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/E1E891B9DFBA47A9A7176D50368E34F7/9986)

Java中，采用的是第二种：共享内存并发模型

## 6.2 JVM运行时数据区

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/6BED1637BD06478098FEB42956A49581/9988)

- 方法区和堆，是对所有线程内存共享的。
- 栈、PC是线程私有的。

Java中内存可见性是指堆中的共享变量

## 6.3 Java内存模型JMM

Java的并发是通过共享内存来实现的。

JMM定义了线程与主内存之间的抽象关系。
- 所有的共享变量都在主内存中
- 每个线程保存一份共享变量的副本

线程之间通信需要以主内存为媒介，而JMM来控制主内存与本地内存的交互：
- 线程对共享变量的操作必须在线程自己的本地内存中进行。
- 线程通信需要一个线程先把线程中的共享变量更新到主内存中
- 然后另一个线程从主内存中读取刷新后的共享变量

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/76D7E6EA6C474B4081B264AFF6EFEF5A/9990)

### 6.3.1 volatile关键字

- 保证多线程操作共享变量的可见性
- 禁止指令重排序

### 6.3.2 JMM与运行时数据区的关系

JMM是抽象的概念。
- JMM的主内存属于共享数据区域，包含堆、方法区
- JMM的本地内存属于私有数据区域，包含程序计数器、本地方法栈、虚拟机栈。