---
title: Iterator
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - java集合框架
tags: 
mathjax: true
katex: true
---
# 1 Iterator
- Iterator类与Enumeration区别
    + 迭代器允许调用者利用定义良好的语义在迭代期间从迭代器所指向的 collection 移除元素，也就是增加了remove方法。
# 2 各个集合的Iterator的实现
1. ArrayList的Iterator实现
它的底层是依赖数组
```
public E next() {
    checkForComodification();
    int i = cursor;    记录索引位置
    if (i = size)    如果获取元素大于集合元素个数，则抛出异常
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i = elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;      cursor + 1
    return (E) elementData[lastRet = i];  lastRet + 1 且返回cursor处元素
}
```
2. HashSet底层是HashMap, HashMap的iterator本质上也是数组。
# 3 fail-fast机制

- 假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException异常，从而产生fail-fast机制。
1. fail-fast解决办法
- 方案一：在遍历过程中所有涉及到modCount值的地方全部加上synchronized或者直接使用Collections.synchronizedList,这种方法不推荐就，因为增删造成的同步锁可能会阻塞遍历操作。
- 方案二：使用CopyOnWriteArrayList替换ArrayList(推荐)
    + 首先修改时加锁
```
public boolean add(E paramE) {
        ReentrantLock localReentrantLock = this.lock;
        localReentrantLock.lock();
        try {
            Object[] arrayOfObject1 = getArray();
            int i = arrayOfObject1.length;
            Object[] arrayOfObject2 = Arrays.copyOf(arrayOfObject1, i + 1);
            arrayOfObject2[i] = paramE;
            setArray(arrayOfObject2);
            int j = 1;
            return j;
        } finally {
            localReentrantLock.unlock();
        }
    }


  
    final void setArray(Object[] paramArrayOfObject) {
        this.array = paramArrayOfObject;
    }
```
    + 其次这三行可以解决很多事情
```
Object[] arrayOfObject2 = Arrays.copyOf(arrayOfObject1, i + 1);
arrayOfObject2[i] = paramE;
setArray(arrayOfObject2);
```
- 在copy的数组上进行修改，完事之后修改引用
- CopyOnWriterArrayList都会copy现有的数据，再在copy的数据上修改，这样就不会影响COWIterator中的数据了，修改完成之后改变原有数据的引用即可。同时这样造成的代价就是产生大量的对象，同时数组的copy也是相当有损耗的。