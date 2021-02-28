---
title: Tomcat 容器启动并加载 DispatcherServlet
date: 2021-2-28
cover:
top_img:
categories: springmvc
tags: 
mathjax: true
katex: true
---
# Tomcat 容器启动并加载 DispatcherServlet 时所做的主要工作

1. 调用 DispatcherServlet 的构造器（当然也会调用父类的构造器，不过构造器默认实现为空；这个动作很短，基本上可以忽略）
2. 调用 GenericServlet 的 init() 方法，不过，这被 HttpServletBean 重写了；同时，重写的 HttpServletBean  的 init() 方法调用了 initServletBean() 方法；而 initServletBean() 方法会完成以下操作：
    - 初始化（创建）一个 WebApplicationContext（实际上是 WebApplicationContext 类）
    - 调用 AbstractApplicationContext 的 refresh() 方法，完成 BeanFactory创建、读取 SpringMVC 配置文件内容、处理容器后处理器、初始化MessageResource、注册监听器等工作
    - 通过上一步中读取到的内容，初始化 HandlerMapping & HandlerAdapter 等工作
    - 上面3个步骤才是重要内容
> 综上：DispatcherServlet 还是一个 Servlet，遵循 constructor() --> init() --> service() --> destroy() 方法的调用流程