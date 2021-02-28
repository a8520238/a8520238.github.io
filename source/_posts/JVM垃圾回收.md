---
title: JVM 垃圾回收
date: 2021-2-28
cover:
top_img:
categories: JVM
tags: 
mathjax: true
katex: true
---
# Java堆内垃圾回收算法 GC

![堆的GC](http://note.youdao.com/yws/public/resource/7839ea220156efcfc3c18b40b088fedd/xmlnote/D70A536610D543B087B07D04B974DE37/2962)
## 1 GC的分类

GC分为
- ==minor GC== ： 发生在 Eden 区满的时候，回收范围为 年轻代。
- ==full GC== ： 发生在老年代满的时候，回收范围是 年轻代 + 老年代。

> 发生 GC 时会产生 STW （Stop the World）机制，暂停 应用线程 的运行，安心进行 GC。而 full GC 会导致 STW 时间更长，因此 JVM 调优就要减少 full GC 的次数。
> 发生 STW 的原因是避免 GC 过程中堆中对象的GC标记（是否为垃圾对象）发生变化。

## 2 堆内对象的存放

堆内的对象

![对象的组成](http://note.youdao.com/yws/public/resource/7839ea220156efcfc3c18b40b088fedd/xmlnote/CC3BFDB10EDE46C9A7BFAD825F3785F3/2966)


## 3 堆内的GC简介

![](http://note.youdao.com/yws/public/resource/7839ea220156efcfc3c18b40b088fedd/xmlnote/E1F7DC7C5EFF45648B02D7DA215FF152/2965)
### 3.1 Minor GC与年轻代

堆是GC的主要场所，年轻代按照容量分为Eden: S_0: S_1 = 8 : 1 : 1。
- Eden区
- Survivor区：存放在GC生存下来的对象，分为两块大小相同的空间：
    + S_0
    + S_1

S_0 和 S_1 又被称为 from 区 和 to 区。
> S_0 和 S_1 总有一个是空的，一个是非空的。当Eden区容量满的时候，会把 Eden 和那个非空的 Survivor 区假设是S0进行 Minor GC ，有引用指向的对象（使用可达性分析算法）会被保留下来并复制到另一个空的 S 区假设是S1，并且分代年龄（分代年龄存在于对象头中）加一，然后 Eden 和 S_0 中所有垃圾对象都会被回收。如此一来，S_0 和 S_1轮流作为非空的 Survivor 区。

#### 3.1.1 担保机制

> 当回收 Eden 和非空的 survivor 区时，发现另一块空的 survivor 区空间不足以存放所有的存活下来的对象，则需要向 Old 区借用空间，称为 担保机制。
> 如果老年区也没有空间了，则触发 Full GC。
> 如果 Full GC 完毕还是不够，则抛出 OutOfMemory 异常。

### 3.2 Full GC 与 老年代

老年代：当年轻代中的对象经过多次 GC 后被保留下来的对象，如果其 分代年龄 到达老年代的要求，则会被放入老年代。

> 老年代满的时候会发生 full gc，回收 老年代 + 年轻代 的所有垃圾对象，当没有可以被回收的对象时，则发生 OOM （outOfMemory） 异常。

## 4 对象何时进入老年代

### 4.1 大对象可以直接进入老年代

这里的大对象指的是需要大量连续内存的对象，比如字符串和数组。可以设置大对象的大小阈值，超过阈值就会直接进入老年代。

这个机制的目的是为了避免为大对象分配内存时的赋值操作而降低效率。

### 4.2 长期存活的对象可以直接进入老年代

虚拟机通过分代收集的思想来管理内存，给每个对象分配一个对象年龄计数器 Age。

- 当对象在 Eden 出生，经过了第一次 Minor GC，存活下来后若能被 Survivor 容纳，则移动到 Survivor 区，并将对象年龄设置为 1。
- 对象每次在 Minor GC 中存活下来，年龄就增长 1。
- 当年龄增长到阈值后被放入老年代。可以通过 -XX:MaxTenuringThreshold 来改变这个阈值。

### 4.3 对象动态年龄判断机制

在那块非空的 Survivor 区，若一批对象的总大小超过了该区内存的 50%, 则此时大于等于该批对象中最大年龄的对象，可以直接进入老年代。

- 该机制是为了让那些可能会长期存活的对象提前进入老年代。
- 该机制的触发一般是在 Minor GC 之后。
- 内存阈值可以调整：-XX:TargetSurvivorRatio

## 5 垃圾回收算法

### 5.1 垃圾判断的可达性分析算法

将 GC Roots 对象为根，向下搜索引用的对象，找到的对象标记为非垃圾对象，存在于堆中但未标记的对象都作为本次垃圾回收的垃圾对象。

- GC Roots 根节点可以是：线程栈的本地变量、静态变量、本地方法栈的变量等。

![可达性算法分析](http://note.youdao.com/yws/public/resource/7839ea220156efcfc3c18b40b088fedd/xmlnote/B44D087D08DE45CB82284F618DBD75C7/2967)

标记-清除算法 （标记后直接清除）

标记-复制算法 （分成两块区域，复制挪动）

标记-整理算法 （标记删除后，零碎内存进行整理）

1 JIT优化，栈上分配，不需要GC。TLAB(Eden中的一块线程私有的空间，Thread Local Allocation Buffer)先创建，不在共享堆中创建，防止同时创建抢夺指针。

2 大对象直接在老年代分配，避免对象来回复制。可以设置大对象的阈值。

3 长期存活的对象进入老年代。

4 动态年龄判断：survivor空间中相同年龄所有对象大小的总和大于survivor空间中的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。

-Xms 初始java堆得大小

jps 查看当前机器运行的java进程

jinfo 进行号 查看详细信息

可视化软件：jvisualvm  visual GC

# 6 慎用finalize()

Finalizable对象的生命周期和普通对象的行为是完全不同的，列举如下：
1. JVM创建Finalizable对象
2. JVM创建 java.lang.ref.Finalizer实例，指向刚创建的对象。
3. java.lang.ref.Finalizer类持有新创建的java.lang.ref.Finalizer的实例。这使得下一次新生代GC无法回收这些对象。
4. 新生代GC无法清空Eden区，因此会将这些对象移到Survivor区或者老生代。
5. 垃圾回收器发现这些对象实现了finalize()方法。因为会把它们添加到java.lang.ref.Finalizer.ReferenceQueue队列中。
6. Finalizer线程会处理这个队列，将里面的对象逐个弹出，并调用它们的finalize()方法。
7. finalize()方法调用完后，Finalizer线程会将引用从Finalizer类中去掉，因此在下一轮GC中，这些对象就可以被回收了。
8. Finalizer线程会和我们的主线程进行竞争，不过由于它的优先级较低，获取到的CPU时间较少，因此它永远也赶不上主线程的步伐。
9. 程序消耗了所有的可用资源，最后抛出OutOfMemoryError异常。