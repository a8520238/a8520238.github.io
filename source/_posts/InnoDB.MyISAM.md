---
title: InnoDB.MyISAM
date: 2021-2-28
cover:
top_img:
categories: 数据库
tags: 
mathjax: true
katex: true
---
# MySQL常见的两种存储引擎：MyISAM与InnoDB

## 1  MyISAM

MyISAM是MySQL的默认数据库引擎（5.5版之前）

- 不支持行锁(MyISAM只有表锁)，读取时对需要读到的所有表加锁，写入时则对表加排他锁；
- 不支持事务
- 不支持外键
- 不支持崩溃后的安全恢复
- 在表有读取查询的同时，支持往表中插入新纪录
- 支持BLOB和TEXT的前500个字符索引，支持全文索引
- 支持延迟更新索引，极大地提升了写入性能
- 对于不会进行修改的表，支持 压缩表 ，极大地减少了磁盘空间的占用

## 2 InnoDB

InnoDB是MySQL的默认数据库引擎（5.5版之后）

- 支持行锁，采用MVCC来支持高并发，有可能死锁
- 支持事务
- 支持外键
- 支持崩溃后的安全恢复
- 不支持全文索引（5.6.4后支持）

## 3 二者对比

1. count运算上的区别： 因为MyISAM缓存有表meta-data（行数等），因此在做COUNT(*)时对于一个结构很好的查询是不需要消耗多少资源的。而对于InnoDB来说，则没有这种缓存。
2. 是否支持事务和崩溃后的安全恢复： MyISAM 强调的是性能，每次查询具有原子性,其执行数度比InnoDB类型更快，但是不提供事务支持。但是InnoDB 提供事务支持事务，外部键等高级数据库功能。 具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。

3. 是否支持外键： MyISAM不支持，而InnoDB支持。