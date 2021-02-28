---
title: volatile
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 8 volatile

## 8.1 为什么存在内存可见性问题

为了提高代码执行速度，CPU先把资源从内存加载到CPU缓存中（L1,L2）等，但操作完并不一定能实现实时写回内存。volatile可以解决这个问题。

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/AD0E00C1F9D64180BA898A8722CCE895/10131)

## 8.2 volatile功能
1. 保证变量的内存可见性
- 当某个线程对`volatile`变量进行写操作时，JMM会立即把线程对应的本地内存刷新到主内存中。
- 当某个线程对`volatile`变量进行读操作时，JMM会立即把该线程的本地内存置为无效，并从主内存中重新读取。
2. 禁止`volatile`变量与普通变量的指令重排
- 禁止指令重排是为了实现一种类似锁的线程间通信机制，但比锁更轻量级。
3. 禁止指令重排示例
- 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
- 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。（这么做的可能原因是volatile读会导致缓存刷新）
- 当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。


### 8.2.1 如何限制指令重排

通过内存屏障来限制指令重排：
- 内存屏障分为读屏障Load Barrier 和 写屏障Store Barrier
- 阻止屏障两侧的指令重排序
- 写后强制把写缓冲区/高速缓存中的脏数据写会主内存，读时先让缓存中数据失效。

**JMM内存屏障**
- 在每个volatile写操作前插入一个StoreStore屏障；
- 在每个volatile写操作后插入一个StoreLoad屏障；
- 在每个volatile读操作后插入一个LoadLoad屏障；
- 在每个volatile读操作后再插入一个LoadStore屏障；

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/70FB36AF392B460397DE6512EDFE1F9F/10133)

- LoadLoad屏障：对于这样的语句Load1,LoadLoad,Load2,在Load2及后续读取操作要读取的数据被访问之前，保证Load1要读取的数据被读取完毕。
- StoreStore屏障：对于这样的语句Store1, StoreStore, Store2,在Store2及后续写入操作执行前，这个屏障会把Store1强制刷新到内存，保证Store1的写入操作对其它处理器可见。
- LoadStore屏障：对于这样的语句Load1,LoadStore,Store2,在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
- StoreLoad屏障：对于这样的语句Store1,StoreLoad,Load2，在Load2及后续所有读取写操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的（冲刷写缓冲器，清空无效化队列）。在大多数处理器的视线中，这个屏障是个万能屏障，兼具其他三种内存屏障的功能。

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/4E78491770DD420F95B5309EEAA41E53/10135)

### 8.2.2 volatile与锁的区别

`volatile`和锁都能保证内存可见性，因此`volatile`可以作为一个轻量级的锁。
- 但`volatile`仅能保证单个变量的读写具有原子性，锁可以保证整个临界区代码的原子性
- 锁的功能更强大，但`volatile`的性能更有优势。
- 对volatile变量的单个对鞋，可以看成是使用同一个监视器锁对这些单个读/写操作做了同步。

## 8.3 volatile实现原理

对`volatile`变量进行写操作时，JVM会向CPU发送带有Lock前缀的指令，该指令会把CPU缓存中的新值写回内存。

同时，缓存一致性协议保证了各个处理器的缓存是一致且最新的。
1. Lock前缀的指令会引起处理器缓存写回内存
2. 一个处理器的缓存回写到内存会导致其他处理器的缓存失效
3. 当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

## 8.4 as-if-serial

含义：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime和处理器都必须遵守as-if-serial语义。为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。
如下：
```
double pi  = 3.14;    //A

　　double r   = 1.0;     //B

　　double area = pi * r * r; //C
```
C依赖A和B，所以A和B一定在C之前发生，但A和B的顺序可以改变。

## 8.5 cpu缓存详解 
> [CPU高速缓存行与内存关系 及并发MESI 协议](https://www.cnblogs.com/jokerjason/p/9584402.html)
> [CPU Cache与缓存行](https://blog.csdn.net/u010983881/article/details/82704733)