---
title: DCL
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# DCL（双重检查加锁）
> 双重检查加锁被熟知为“懒汉式”单列模式的实现
其代码如下：
```
public class DoubleCheckedLocking {
    
    private static SingleInstance singleInstance;

    public static SingleInstance getInstance() {
        if (singleInstance == null) {
            synchronized (DoubleCheckedLocking.class) {
                if (singleInstance == null) {
                    singleInstance == new SingleInstance();
                }
            }
        }
    }
}
```

1. DCL主要是为了使用延迟初始化来降低“饿汉式”单例模式对JVM启动时的性能影响。
2. 之所以两次判断singleInstance是否为null,是因为存在无序写入，导致singleInstance引用已对其它线程可见，但对象内容singleInstance.getId() 对其它线程并不可见。就是因为对象构造过程中一系列指令写入内存的乱序，导致了失效对象的产生。
3. 可以使用valatile变量解决这个问题，但现在多用枚举创建。