---
title: BeanPostProcess
date: 2021-2-28
cover:
top_img:
categories: spring
tags: 
mathjax: true
katex: true
---
![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/AF2DB6285CB84DFABD4E33231C051A9F/3688)





- 过程： 构造 -> 注入 -> postprocessBeforeInitialization -> InitializingBean -> postprocessAfterInitialization

- Object postprocessBeforeInitialization(Object bean, String beanName)

其中bean就是创建的对象，beanName是对象id，获得创建好的对象。

在实战中很少使用初始化操作。那么只要取上述方法之一即可， 一般是after。