---
title: JVM CLass文件
date: 2021-2-28
cover:
top_img:
categories: JVM
tags: 
mathjax: true
katex: true
---
class 文件

第一部分:
CAFEBABE 前四个字节 魔数

第二部分：次版本号
0000 （0）

第三部分：0034（52） 51 是 1.7

第四部分：常量池的个数 2个字节

个数-1，第0位被jvm引用代表深，什么都不引用

常量池分为 字面量和符号引用类型，第一个字节是tag位，class常量池类型分类表。


class文件表全集  jad反编译
jclasslib插件