---
title: ArrayList
date: 2021-2-28
cover:
top_img:
categories: java集合框架
tags: 
mathjax: true
katex: true
---
# 1 ArrayList的优缺点
## 1.1 优点
1. ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找也就是get的时候非常快
2. ArrayList在顺序添加一个元素的时候非常方便，只是向数组里面添加了一个元素
## 1.2 缺点
1. 删除元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能
2. 插入元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能
## 1.3 总结
ArrayList比较适合顺序添加、随机访问的场景
# 2 ArrayList线程安全
1. 一个方法时Collections.synchronizedList方法把你的ArrayList变成一个线程安全的List,如
```
List<String> synchronizedList = Collections.synchronizedList(list);
synchronizedList.add("aaa");
synchronizedList.add("bbb");
for (int i = 0; i < synchronizedList.size(); i++)
{
    System.out.println(synchronizedList.get(i));
}
```
2. Vector弃用原因
- 所有get() ， set()方法都是synchronized ，因此您无法对同步进行细粒度控制。
- 而ArrayList虽然不是线程安全的就，但是可以对整个过程进行加锁。

> Vector 使用同步方法实现，synchronizedList
使用同步代码块实现。

# 3 RandomAccess接口
1. RandomAccess接口是空的，其作用是标记，标记实现其的class底层是数组，可以用for遍历
2. 经常会用instanceof来判断List集合子类是否实现RandomAccess接口
## 3.1 Arraylist用for循环遍历比iterator迭代器遍历快，LinkedList用iterator迭代器遍历比for循环遍历快

原因： 
- LinkedList底层是双向链表，用for循环遍历的话，get(10)会从头获取，浪费时间，所以迭代器快

# 4 ArrayList 使用了transient 关键字进行存储优化，而Vector 没有

ArrayList 实现了writeObject 方法，可以看到只保存了非null 的数组位置上的数据。
即list 的size 个数的elementData。需要额外注意的一点是，ArrayList 的实现，提供了
fast-fail 机制，可以提供弱一致性。

Vector 也实现了writeObject 方法，但方法并没有像ArrayList 一样进行优化存储，实现语句是：
`data = elementData.clone();`,

clone()的时候会把null 值也拷贝。所以保存相同内容的Vector 与ArrayList，
Vector 的占用的字节比ArrayList 要多。

# 5 ArrayList（Collections.sychronized）和Vector主要区别（Stack继承Vector）
1. 数组扩容：ArrayList扩容1.5， Vector扩容2
2. 同步代码块和同步方法的区别
    - 同步代码块在锁定的范围上可能比同步方法要小，一般来说锁的范围大小和性能是
成反比的。
    - 同步块可以更加精确的控制锁的作用域（锁的作用域就是从锁被获取到其被释放的
时间），同步方法的锁的作用域就是整个方法。
    - 静态代码块可以选择对哪个对象加锁，但是静态方法只能给this 对象加锁。
3. SynchronizedList 有
很好的扩展和兼容功能。他可以将所有的L i s t 的子类转成线程安全的类
4. 使用
SynchronizedList 的时候，进行遍历时要手动进行同步处理
5. SynchronizedList 可以
指定锁定的对象。