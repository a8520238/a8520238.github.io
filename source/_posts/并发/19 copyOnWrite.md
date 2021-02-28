---
title: copyonwrite
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 19.1 CopyOnWrite
1. CopyOnWrite机制
- CopyOnWrite容器即是写时复制的容器， 当向一个容器中添加元素的时候，不直接往容器中添加，而是将当前容器进行copy，复制出来一个新的容器，然后向新容器中添加我们需要的元素，最后将原容器的引用指向新容器。

2. CopyOnWriteArrayList
- 优点：读写分离，无需同步措施
- 缺点：
    + CopyOnWriteArrayList每次执行写操作都会将原容器进行拷贝一份
    + CopyOnWriteArrayList写和读分别作用在不同的新老容器，写操作的过程中，读不会阻塞，但读到的却是老容器的数据。