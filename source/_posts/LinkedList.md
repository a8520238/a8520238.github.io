---
title: LinkedList
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
# 1 LinkedList

- LinkedList与ArrayList一样实现List接口，只是ArrayList是List接口的大小可变数组的实现。
- 基于链表实现的方式使得LinkedList在插入和删除时更优于ArrayList,而随机访问则慢一些。
- LinkedList实现所有可选的列表操作，并允许所有的元素包括null
- 除了实现List接口外，LinkedList类还为在列表的开头及结尾get、remove和insert元素提供了统一的命名方法。这些操作允许将链接列表用作堆栈、队列或双端队列。
- 此类还实现了Deque接口，为add、poll提供先进先出队列操作，以及其他堆栈和双端队列操作。

1. LinkedList 的 get和add方法都会调用node方法，该方法会判断节点距离头近还是尾近，选择从哪一端遍历
2. linkedlist中的push add是在尾部插入，push是在头部插入，poll,pop是在头部删除

# 2 Queue <-- Deque -- < LinkedList/ArrayDeque
1. Deque继承了Queue接口，创建双向队列，队头和队尾均可以插入或删除，主要实现类是ArrayDeque和LinkedList

## 2.1 ArrayDeque（底层使用循环数组实现双向队列）
1. add方法会调用addLast方法, 与linkedlist相似
2. 循环数组的判定方法
```
// 尾索引+1，并与数组（length - 1）进行取‘&’运算，因为length是2的幂，所以（length-1）转换为2进制全是1，
   // 所以如果尾索引值 tail 小于等于（length - 1），那么‘&’运算后仍为 tail 本身；如果刚好比（length - 1）大1时，
   // ‘&’运算后 tail 便为0（即回到了数组初始位置）。正是通过与（length - 1）进行取‘&’运算来实现数组的双向循环。
   // 如果尾索引和头索引重合了，说明数组满了，进行扩容。
   if ((tail = (tail + 1) & (elements.length - 1)) == head)
      doubleCapacity();// 扩容为原来的2倍
```
```
addFirst(E e) 的实现：
public void addFirst(E e) {
   if (e == null)
      throw new NullPointerException("e == null");
   // 此处如果head为0，则-1（1111 1111 1111 1111 1111 1111 1111 1111）与（length - 1）进行取‘&’运算，结果必然是（length - 1），即回到了数组的尾部。
   elements[head = (head - 1) & (elements.length - 1)] = e;
   // 如果尾索引和头索引重合了，说明数组满了，进行扩容
   if (head == tail)
      doubleCapacity();
}
```
> 也就是说ArrayDeque中有效部分是头结点到尾结点的部分

# 3 PriorityQueue(底层用数组实现堆的结构)

- 默认是小顶堆
- 并发使用优先组队列(PriorityBlockingQueue)
- 调整包括从上往下调整和从下往上调整，在remove方法中体现尤为重要
```
public boolean remove(Object o) {
    int i = indexOf(o); // 找到数据对应的索引
    if (i == -1) // 不存在的话返回false
        return false;
    else { // 存在的话调用removeAt方法，返回true
        removeAt(i);
        return true;
    }
}
private E removeAt(int i) {
    modCount++;
    int s = --size; // 元素个数-1
    if (s == i) // 如果是删除最后一个叶子节点
        queue[i] = null; // 直接置空，删除即可，堆还是保持特质，不需要调整
    else { // 如果是删除的不是最后一个叶子节点
        E moved = (E) queue[s]; // 获得最后1个叶子节点元素
        queue[s] = null; // 最后1个叶子节点置空
        siftDown(i, moved); // 从上往下调整
        if (queue[i] == moved) { // 如果从上往下调整完毕之后发现元素位置没变，从下往上调整
            siftUp(i, moved); // 从下往上调整
            if (queue[i] != moved)
                return moved;
        }
    }
    return null;
}
```
1.  siftDown从上向下调整
```
siftDown(i, moved); // 从上往下调整
 private void siftDownUsingComparator(int k, E x) {
        int half = size >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((E) c, (E) queue[right]) > 0)
                c = queue[child = right];
            if (comparator.compare(x, (E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = x;
    }
```
2. siftup从下向上调整
```
siftUp(i, moved); // 从下往上调整

    private void siftUpUsingComparator(int k, E x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (comparator.compare(x, (E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }
```
- add时的上滤
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/13323E2C62D44B1CB9A28FA1C0D6D38D/12648)
- poll时的下滤
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/1451B332D1D0487EBCEA24B9A4A92F64/12652)
- 在中间删除时的下滤和上滤
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/62C9B8F6F7F743D190C3A9A91F934192/12654)
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/71D0F704FAE04DE9BC50FE9B179D2AAF/12656)