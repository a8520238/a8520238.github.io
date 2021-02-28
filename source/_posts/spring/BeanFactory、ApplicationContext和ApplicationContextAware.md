---
title: BeanFactory ApplicationContext ApplicationContextAware
date: 2021-2-28
cover:
top_img:
categories: spring
tags: 
mathjax: true
katex: true
---
# BeanFactory

- BeanFactory负责读取bean配置文件，管理bean的加载，实例化，维护bean之间的依赖关系，负责bean的声明周期。


# ApplicationContext
- ApplicationContext处理提供BeanFactory的功能外，还提供了更完整的框架功能。
- 常用的获取ApplicationContext的方法：
    + FileSystemXmlApplicationContext：从文件系统或者url指定的xml配置文件创建，参数为配置文件名或文件名数组
    + ClassPathXmlApplicationContext：从classpath的xml配置文件创建，可以从jar包中读取配置文件
    + WebApplicationContextUtils：从web应用的根目录读取配置文件，需要先在web.xml中配置，可以配置监听器或者servlet来实现
```
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<servlet>
<servlet-name>context</servlet-name>
<servlet-class>org.springframework.web.context.ContextLoaderServlet</servlet-class>
<load-on-startup>1</load-on-startup>
</servlet>
```
这两种方式都默认配置文件为web-inf/applicationContext.xml，也可使用context-param指定配置文件
```
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>/WEB-INF/myApplicationContext.xml</param-value>
</context-param>
```
# ApplicationContextAware

当一个类实现了这个接口（ApplicationContextAware）之后，这个类就可以方便获得ApplicationContext中的所有bean。换句话说，就是这个类可以直接获取spring配置文件中，所有有引用到的bean对象。