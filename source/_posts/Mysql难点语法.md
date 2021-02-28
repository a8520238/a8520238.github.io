---
title: mysql难点语法
date: 2021-2-28
cover:
top_img:
categories: 数据库
tags: 
mathjax: true
katex: true
---
# Mysql难点语法

## 1 group by

group by 要与having共用， 不能与where，因为where中不能有聚集函数做为条件表达式

having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数

## 2 数据库中的union与union all的区别

Union因为要进行重复值扫描，所以效率低。如果合并没有刻意要删除重复行，那么就使用Union All

 两个要联合的SQL语句 字段个数必须一样，而且字段类型要“相容”（一致）；
 
 union和union all的区别是,union会自动压缩多个结果集合中的重复结果，而union all则将所有的结果全部显示出来，不管是不是重复。 
 
 Union All：对两个结果集进行并集操作，包括重复行，不进行排序； 
 
 ## 3 连表查询比子查询快
 
 ## 4 按某一属性排序并生成行号序列
 ```
 row_number() over(partition by customer_id 
 order by order_date desc)
 ```
 ## 5 using连接查询
 
 sql/92标准可以使用using关键字来简化连接查询，但是只是在查询满足下面两个条件时，才能使

用using关键字进行简化。
- 1.查询必须是等值连接。
- 2.等值连接中的列必须具有相同的名称和数据类型。

## 6 case when

```
    case
        when ...
        whem ...
        else ...
    end
```