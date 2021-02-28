---
title: mysql初学笔记
date: 2021-2-28
cover:
top_img:
categories: 数据库
tags: 
mathjax: true
katex: true
---
# mysql笔记 （关系型数据库）

## 一 使用终端操纵数据库

### 1. 登录数据库
mysql -uroot -p
### 2. 查询所有数据库
show database;
### 3. 选中某一数据库操作
use database;
### 4.查询 
select * from admin where admin_Id = 1
### 5.退出
exit;或quit;
### 6.创建数据库
create database name;
### 7.查看数据库中的数据表
show tables;
### 8.创建数据表
创建宠物数据表

    CREATE TABLE pet (
    name VARCHAR(20),
    owner VARCHAR(20),
    sex CHAR(1),
    birth DATE);
    
### 9. 查看创建好的数据表
decribe pet;

### 10 查看表中记录
select * from pet;

### 11.插入记录
    INSERT INFO pet
    VALUES ('pp','mm','f','1999-03-26');
### 12.mysql 常用数据类型
数值
类型|大小|范围（有符号|范围(无符号)|用途
-|:-:|:-:|:-:|:-:|-

日期/时间

字符串

数据类型选择 日期按照格式 数值和字符串按照大小
### 13. 删除数据
delete from pet where name ='pp';
### 14.修改数据
update pet set name='旺财' where owner = 'mm';
### 15.mysql建表约束条件

1.主键约束

    id int(11) primary key

它能够唯一确定一张表中的一条记录，也就是我们通过给某个字段添加约束就可以使该字段不重复且不为空。

可以有多个主键，主键联合约束。只要联合的主键值加起来不同即可。但联合主键每一个都不能为空。

2. 自增约束

    primary key auto_increment
与主键约束搭配起来进行自增

忘记添加主键，创建后添加

    alter table user4 add primary key(id);

输出表结构主键

    alter table user4 drop primary key;
    
修改字段主键约束

    alter table user4 modify id int primary key;
    
    
3. 外键约束

[有主从表|主外键关系时删除表和删除数据](https://blog.csdn.net/wdd199801140310/article/details/102984378)

涉及到两个表，一个是主表，一个是副标，或父表，子表，外键设置在副表上

    create table classes(
        id int primary key,
        name varchar(20)
    );

    create table students(
        id int primary key,
        name varchar(20),
        class_id int,
        foreign key(class_id) references classes(id)
    );

副表中的外键值要参照主表， 1.主表中没有的数据值，副表不能使用。2.被副表引用的主表值不允许被删除。


4. 唯一约束

约束修饰的字段的值不可以重复

    name carchar(20) unique
    alter table user add unique(name); 
    
可以unique(name, id)
这两个件在一起不重复就可以。

删除唯一约束

    alter table user drop index name
    
modify 添加

    alter table user modify name varchar(20 unique);
    

5. 非空约束

修饰的字段不能为空

    create table user (
        id int,
        name varchar(20) not null
    );
    
单独插入会报错

    insert into user (name) values 'list';
    
    
6. 默认约束

插入字段时，如果没有传值，使用默认值

    age int default 20

### 16.数据库设计三大设计范式

1.第一范式 1NF

数据表中的所有字段都是不可分割的原子值

    create table user (
        id int primary key,
        name varchar(20),
        address varchar(30)
    );

字段值还可以继续拆分， 就不满足第一范式, 上表不满足第一范式

    create table user (
        id int primary key,
        name varchar(20),
        cuntry varchar(30),
        province varchar(30),
        city varchar(30)
    );
    
上表拆分比较详细，满足第一范式。

范式设计一般来说越详细越好，但还是要根据具体工程。

2. 第二范式

必须是满足第一范式的前提下，第二范式要求，除主键外的每一列都必须完全依赖主键

如果出现不完全依赖，只可能发生在联合主键的情况下。某字段只依赖其中一个主键。

如果实在要这样，设计成外键关联，拆分表。

3.第三范式 3NF

必须先满足第二范式，除开主键列的其他列之间不能有传递依赖。不能有冗余依赖。需要拆分。

### 17.查询联系

学生表
Student
学号
姓名
性别
出生年月日
所在班级

    create table student {
        sno varchar(20) primary key,
        sname varcher(20) not null,
        ssex varchar(10) not null,
        sbirthday datatime,
        class varchar(20)
    }

成绩表
Score
学号
课程号
成绩

    create table score(
        sno varchar(20) not null,
        cno varchar(20) not null,
        degree decimal,
        foreign key(sno) references student(sno),
        foreign key(cno) references course(cno),
        primary key(sno, cno)
    );
    
查询所有记录 select * from student;

查询指定记录，（某些字段）， select sname, ssex, class from student;

查询不重复的depart列
select distinct depart from teacher;

查询区间
select * from score where degree between 60 and 80;
或者运算符比较
select * from score where degree > 60 and degree < 80;

查询或关系
select * from score where degree in (85, 86, 88);

查询表中性别为女的同学记录
select * from student where class='95031' or ssex='女';

以class降序查询
select * from student order by class desc;
默认升序或者arc

以cno升序，degree 降序查询所有记录
select * from score order by cno asc, degree desc;

查询总人数 count
select count(*) from student where class='95031';

查询最高分
select sno,cno from score where degree=(selectmax(degree) from score);

查询每门课的平均成绩

select avg(degree) from score where cno='3-105'

select cno,avg(degree) from score group by cno;

查询score表中至少有两名学生选修的并以3开头的课程

select cno, avg(degree),count(*)) from score group by  cno having count(cno)>=2 and cno like '3%';

查询分数大于70小于90的sno列

select sno,degree from score where degree>70 and degree < 90;
或者用between and

多表查询

查询所有学生的sname, cno和degree

select sname,con,degree from student,score where student.sno=score.sno;

三表关联查询(重点)
通过字段相等联查

select sname,cname,degree from student,course,score where student.sno=score.sno and course.cno=score.cno;

查询'95031'班学生每门课的平均分

select cno,avg(degree) from score where sno in (select sno from student where class='95031') group by cno;

18. 查询"3-105"课程的成绩高于"109"号同学 "3-105"成绩的所有同学的记录。

select * from score where cno= '3-105' and degree > (select degree from score where sno='109' and cno='3-105');

查询和学号为108、101的同学同年出生的所有学生的sno\sname和sbirthday列

select * from student where year(sbirthday) in (select year(sbirthday) from student where sno in (108, 101));

查询 "张旭" 教师任课的学生成绩

select * from score where cno =(select cno from course where tno= (select * from teacger where tname= '张旭'));

查询选修某课程的同学人数多于5人的教师姓名。

select tname from teacher where tno = (select tno from course where cno = (select cno from score groupby cno having count(*) > 5;));

查询95033班和95031班全体学生的记录。

select * from student where class in ('95031', '95033');

查询存在成绩在85分以上的课程的cno

select * from score where degree>85;

查询出 "计算机系" 教师所有课程的成绩表

select * from score where cno in (select cno from course where tno in (select tno from teacher where depart= '计算机系';));

查询 "计算机系" 与 "电子工程系"不同职称的教师的tnam和prof

union 求并集

select * from teacher where depart = '计算机系' and prof not in (select prof from teacher where depart = '电子工程系')
union
select * from teacher where depart = '电子工程系' and prof not in (select prof from teacher where depart = '计算机系')

查询选修编号为 "3-105"课程且成绩至少高于选修课程 "3-245"的同学的cno,sno和degree,并且按degree从高到低排序

select * from score where cno = '3-105' and degree > any(select degree from score where cno = '3-245') order by degree desc;

查询选修编号为 "3-105"
且成绩高于选修编号为 "3-245" 的同学的 cnp\sno 和 degree

select * from score where cno= '3-105' and degree>all(select degree from score where cno='3-245');

查询所有教师和同学的name,sex和birthday

select tname as name,tsex as sex,tbirthday as birthday from teacher
union
select sname,ssex,sbirthday from student

查询成绩比该课程平均成绩低的同学的成绩表

select * from score a where degree < (select avg(degree) from score b where a.cno=b.cno);

查询至少有两名男生的班号

select class from student where ssex= '男' group by class having count(*)>1;

查询student 表中最大值和最小sbirthday日期值

select max(sbirthday) as '最大'，min(sbirthday) as '最小' from student;

### 18 SQL的四中连接查询

内连接
inner join 或者 join

外连接
1.左连接 left join 或者 left outer join

2.右连接 right join 或 outer join

3.完全外连接 full join 或者 full outer join

    create table person(
        id int,
        name varchar(20),
        cardId int
    );

    create table card(
        id int,
        name varchar(20),
    );

insert into person values();

inner join查询
select * from person inner join card on person.cardId = card.id;

内联查询，就是两章表中的数据，通过某个字段相对，查询出相关记录数据。

left join（左外连接）

select * from person left join card on person.cardId=card.id;

左外连接吧左边表里的所有数据取出来，右边表有相等显示出来，没有就会补null

right join（右外连接）

select * from person right (outer) join card on person.cardId=card.id;

同左外连接，左右互换

full join(全外链接)

select * from person full join card on person.cardId=card.id;

mysql不支持full join
求出两边表的并集

mysql通过union实现

select * from person left join card on person.cardId=card.id
union
select * from person right join card on person.cardId=card.id;

### 20.mysql 事务

mysql中，事务湿气重一个最小的不可分割的单元，事务能够保证一个业务的完整性。

多条mysql同时成功或者同时失败的要求。

mysql是默认开起事务的

默认事务开启的作用

当我们去执行一个sql语句时，效果立刻体现，不会回滚。

回滚是rollback；

设置自动提交为false
set autocommit=0;

begin或者start transaction 开启一个事务，之后可以回滚
直到commit

事务的四大特征

A.原子性：事务是最小单位，不可以分割

C.一致性：事务要求，同一事物中的sql语句必须同时成功或失败

I。隔离性：事务1和事务2之间具有隔离性。

D.持久性：事物一旦结束（commit,rollback），就不可以返回。

事务开启

1.修改默认提交 set autocommit=0;

2.begin

3.start transaction

事务的隔离性

1.read uncommited (读未提交)

事务对数据进行操作，在操作过程中，事务没有被提交，但是b可以看见a操作的结果。

查看隔离级别mysql8.0

系统级别

select @@global.transaction_isolation;

会话级别

select @@transaction_isolation

设置

set global transaction isolation level read uncommited;

这种读未提交的叫做脏读

2.read commited
(读已经提交)

3.repeatable read
(可以重复读)

4.serializable
(串行化)


## 二 使用可视化工具操纵数据库



## 三 在编程语言中使用数据库