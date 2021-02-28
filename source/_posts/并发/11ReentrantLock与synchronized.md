---
title: ReentrantLock与synchronized的区别和原理
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 11 ReentrantLock与synchronized的区别和原理

## 11.1 二者的相同点
1. 两个都是可重入锁
2. 都是阻塞式同步
- 线程的阻塞和唤醒的代价是比较高的（操作系统需要在用户态与内核态之间来回切换，代价很高，可以通过锁优化改善）
3. 都是独占锁

> 读写锁ReentrantReadWriteLock中，读锁是共享锁，写锁是排他锁

## 11.2 二者的不同点

1. 原始构成
- synchronized是java语言的关键字，是原生语法层面的互斥，需要jvm实现。
- ReentrantLock是JDK1.5之后提供的API层面的互斥锁类
2. 实现
- synchronized通过JVM加锁解锁，lock通过api层面的加锁解锁，需要手动释放锁。
3. 灵活性
- synchronized的范围是整个方法或synchronized块部分
- lock是方法调用，可以跨方法，灵活性大
4. 等待中可中断
- synchronized不可中断，除非抛出异常。释放锁的方式有两个：1是代码执行完，正常释放锁；2是抛出异常，由JVM退出等待。
- 持有lock的线程长期不释放的时候，正在等待的线程可以选择放弃等待
    + 设置超时方法 tryLock(long timeout, TimeUnit unit)，时间过了就放弃等待；
    + lockInterruptibly()放代码块中，调用interrupt()方法可中断，而synchronized不行)
5. 是否是公平锁
- synchronized是是公平锁
- lock可以选择是否是公平锁，非公平false,公平true
6. 性能
- 资源不是很激烈的情况，synchronized很适合，ReentrantLock性能差一些。但在同步非常激烈的情况下，synchronized的性能一下子下降几十倍，而lock几乎不变。

## 11.3 synchronized 原理

### 11.3.1 monitor
synchronized进行编译，会在同步块的前后分别形成`monitorenter`和`monitorexit`这两个字节码指令。

每个对象都有一个监视器锁（monitor）

在执行`monitorenter`指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1.

在执行`monitorexit`指令时会将锁计算器减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程会阻塞。
### 11.3.2 锁释放和锁获取的内存语义
锁释放和锁获取的内存语义与volatile有相同的内存语义
1. 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的消息）。
2. 线程B获取一个锁，实质上是线程B接受了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。
3. 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息

### 11.3.3 实现原理
- 实质
1. 普通同步方法， 锁是当前实例对象
2. 静态同步方法， 锁是当前类的class对象
3. 同步方法块，锁是括号里的对象
- java对象头、monitor是synchronized的基础
- synchronized底层是操作系统的mutex lock调用，需要进行用户态到内核态的切换，开销比较大。

[synchronized底层](https://blog.csdn.net/niuwei22007/article/details/51433669)

### 11.3.4 synchronized修饰普通方法和静态方法
1. synchronized修饰普通方法
- 当该类中有多个普通方法被Synchronized修饰（同步），那么这些方法的锁都是这个类的一个对象this.
- 调用同一对象的不同方法会阻塞
2. synchronized修饰静态方法
- 锁是类锁(.class)。这个范围就比对象锁大。这里就算是不同对象，但是只要是该类的对象，就使用的是同一把锁。多个线程调用该类的同步的静态方法时，都会阻塞。
> synchronized修饰普通方法，实际上是对this对象加锁，修饰静态方法，是对.class对象进行加锁

## 11.4 ReentrantLock原理

ReentrantLock是juc包下提供的一套互斥锁，相比synchronized，reentrantlock提供以下高级功能：
1. 等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相对于synchronized来说可以避免出现死锁。通过lock.lockInterruptibly()来实现这个机制。
2. 可以实现公平锁
- 创建非公平锁
```
Lock lock = new ReentrantLock();
Lock lock = new ReentrantLock(false)
```
- 创建公平锁
```
trueLock lock = new ReentrantLock(true);
```
3. 锁绑定多个条件，一个ReentrantLock对象可以同时绑定多个对象。
- ReenTrantLock提供了一个Condition类，用来实现分组唤醒需要唤醒的线程门，而不是像synchronized要么随机唤醒一个线程要么唤醒所有线程。