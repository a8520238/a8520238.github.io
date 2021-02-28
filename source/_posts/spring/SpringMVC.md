---
title: springmvc
date: 2021-2-28
cover:
top_img:
categories: springmvc
tags: 
mathjax: true
katex: true
---
# SpringMVC


## 1 MVC作用

1. MVC框架提供了控制器Controller调用Service DAO
2. 请求响应的处理
3. 接受请求参数 request.getParameter("")
4. 控制程序的运行过程
5. 视图解析

## 2 Spring整合MVC框架的核心思路

- 准备工厂
    + 创建工厂```ApplicationContext ctx = new WebXmlApplicationContext()```
    + 如何保证唯一被共用：
       工厂被存储在ServletContext这个人作用域中， 再创建ServletContext对象，创建的同时，创建工厂，存储在```ServletContext.setAttribute("xxx", ctx)```;
       ```ServletContextListener```在Servlet对象创建中，只会被调用一次，把工厂创建的代码写在其中即可。
    + Spring封装了一个ContextLoaderListener,包括两个功能，一个是创建工厂，第二个是把工厂存在ServletContext中，也就是上述过程。
    ![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/13477688943E4C008930B0DC3531C382/4259)
- 代码整合
    ![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/2668940F0E494A89B88B9F8A09A3C46B/4265)