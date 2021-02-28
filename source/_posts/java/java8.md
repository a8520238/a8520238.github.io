---
title: java8
date: 2021-2-28
cover:
top_img:
categories: javaSE
tags: 
mathjax: true
katex: true
---
# java8新特性

主要两个 stream lamba

这篇博客不错

https://blog.csdn.net/weixin_42210904/article/details/91987891

## 1 Function.identity()是什么

Function是一个接口，那么Function.identity()是什么意思呢？解释如下：

Java 8允许在接口中加入具体方法。接口中的具体方法有两种，default方法和static方法，identity()就是Function接口的一个静态方法。
Function.identity()返回一个输出跟输入一样的Lambda表达式对象，等价于形如t -> t形式的Lambda表达式。

identity() 方法JDK源码如下：

    static  Function identity() {
        return t -> t;
    }
    
## 2 stream生成
- 调用 stream.of
```
Stream<String> stream = Stream.of("Hollis", "HollisChuang", "hollis", "Hello", "HelloWorld", "Hollis");
```
- Arrays.stream

- list引用.stream
```
List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis", "Hello", "HelloWorld", "Hollis");
Stream<String> stream = strings.stream();
```

## 3 数组转字符串

```
String str1 = Arrays.stream(arr).boxed().map(i -> i.toString()) //必须将普通数组 boxed才能 在 map 里面 toString
				.collect(Collectors.joining(""));
```

##  4 数组转字符串数组

```
String[] arr = Arrays.stream(nums).boxed().map(aa -> aa.toString()).toArray(String[]::new);

//变成list .collect(Collectors.toList());
```
