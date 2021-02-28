---
title: mysql底层架构
date: 2021-2-28
cover:
top_img:
categories: 数据库
tags: 
mathjax: true
katex: true
---
# 1 MySQL架构
[Mysql底层架构](https://www.itzhai.com/articles/insight-into-the-underlying-architecture-of-mysql-buffer-and-disk.html)

[函数sync、fsync与fdatasync的总结整理](https://www.jb51.net/article/101062.htm)

[覆盖索引与回表](https://www.jianshu.com/p/8991cbca3854)

[Doublewrite Buffer与redo log](https://www.cnblogs.com/geaozhang/p/7241744.html)

[InnoDB事务日志（redo log 和 undo log）详解](https://www.cnblogs.com/better-farther-world2099/p/9290966.html)
1. 相关概念：
    - 内存结构： buffer pool、log buffer、change buffer, buffer pool的页淘汰机制是怎样的
    - 磁盘结构：系统表结构、独立表空间、通用表空间、undo表空间、redo log;

2. 为什么MySQL使用B+树？
    - 哈希表虽然可以提供O(1)的单行数据操作性能，但却不能很好的支持排序和范围查找，会导致全表扫描；
    - B树可以再非叶子节点存储数据，但是这可能会导致查询连续数据的时候增加更多的I/O操作；
    - 而B+树数据都存放在叶子节点，叶子节点通过指针相互连接，可以减少顺序遍历时产生的额外随机I/O
3. 为什么使用自增主键
    - 自增主键的插入是递增顺序插入的，每次添加记录都是追加的，不涉及到记录的挪动，不会触发叶子节点的分裂，而一般业务字段做主键，往往都不是有序插入的，写成本比较高，所以我们更倾向于使用自增字段作为主键。
4. 聚集索引
    - 当在表上面定义了PRIMARY KEY之后，InnoDB会把它作为聚集索引。
    - 如果您没有为表定义PRIMARY KEY，则MySQL会找到第一个不带null值的UNIQUE索引，并其用作聚集索引；
    - 如果表没有PRIMARY KEY或没有合适的UNIQUE索引，则InnoDB 内部会生成一个隐藏的聚集索引GEN_CLUST_INDEX，作为行ID，行ID是一个6字节的字段，随着数据的插入而自增。
