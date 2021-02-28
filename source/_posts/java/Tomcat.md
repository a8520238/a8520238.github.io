---
title: tomcat
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - javaEE
tags: 
mathjax: true
katex: true
---
# Tomcat

> Tomcat的默认端口是8080,如果想要修改，需要在server.xml修改<Connector>下的路由。

> Tomcat服务器的根目录下有一个logs文件夹，其中有一个以"catalina.yyyy-MM-dd.log"形式命名的log文件，例如"catalina.2014-05-17.log"日志文件就是记录Tomcat服务器2014-05-17这一天的运行情况。

## 1 抽象化Tomcat
```
class Tomcat {
    Connector connector;
    List<Servlet> servlets;
} 
Servlet ==== server + applet(客户端);

Java applet -- 客户端；

Java servlet -- 服务端 网络请求；
```
- tomcat 需要实现HttpServletRequest 做为doGet的参数

tomcat服务器的启动是基于一个server.xml文件的，Tomcat启动的时候首先会启动一个Server，Server里面就会启动Service，Service里面就会启动多个"Connector(连接器)"，每一个连接器都在等待客户机的连接，当有用户使用浏览器去访问服务器上面的web资源时，首先是连接到Connector(连接器)，Connector(连接器)是不处理用户的请求的，而是将用户的请求交给一个Engine(引擎)去处理，Engine(引擎)接收到请求后就会解析用户想要访问的Host，然后将请求交给相应的Host，Host收到请求后就会解析出用户想要访问这个Host下面的哪一个Web应用,一个web应用对应一个Context。
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/0B563EA3F89144F59D3B673D18D41EC4/6758)

## 2 tomcat包括
### 2.1 tomcat包括

![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/B53EDE320F794119B89990BECC0A6FAA/6391)

### 2.2 tomcat配置

服务器核心配置文件在conf/server.xml

### 2.3 修改localhost

在server.xml中修改<host>标签

然后再driver/etc/hosts修改127.0.0.1对应我们新修改后的主机名

### 2.4 tomcat web目录
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/458BBBDDE9284E6D9940F19E569FA882/6789)

## 3 tomcat架构和底层原理

![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/3D935FBB9F9045DAB53EE940D011499D/7264)

### 3.1 Tomcat是一个Servlet容器
```
class Tomcat {
    Connector connector;
    List<Servlet> servlets;
} 
```
1. HttpServletRequest 是Tomcat实现的类RequestFacade(门面模式)
2. MyServlet extends HttpServlet 是tomcat依据class文件创建的。

> 门面模式：（相当于1对多的代理模式）

>    RequestFacde 继承HttpServletRequest

>    Request 也继承HttpServletRequest

> RequestFacde会调用Request中的同名方法   

3. Tomcat部署的三种方法
- 文件夹部署 把class文件夹拉入，换成.war
- war包部署 把war包拉入
- Context部署 <Context path="/ServletHello(项目名(也就是war包|文件夹的名字))" docBase="实际路径"/>

4. context 
- context本质也是一个Servlet容器 它继承了Container

5. context Host Wrapper Engine
- 这四个接口是Tomcat的四个容器
- Context 应用--servlet(存放)
- Host 虚拟主机--Context(存放),对应localhost那一层的名字
- Engine Host容器，用来管理主机集群
- Wrapper
    + Engine->Host->Context->Wrapper->Servlet
    + 对每一个HttpServlet进行单例创建，也就是每个Servlet类有多个Servlet实例，Wrapper用来存放每个类的实例集合。
    + Wrapper -> List<Servlet> servlets;
    + Context -> List<Wrapper> list;

6. Pipeline

上述四个容器，每个容器下都有一个PipeLine.管道中有阀门（Valve），将请求传递给下一层容器。

[语雀周瑜优质资源](https://www.yuque.com/renyong-jmovm/kb)

### 3.2 Request对象生成

> 端口和应用程序绑定

1. 服务器A和服务器B之间的通讯由TCP可靠
2. TCP由操作系统实现
    + linux: tcp_connect() //建立TCP连接
    + 负责数据可靠传输
3. HTTP可由Tomcat|浏览器实现
    + 浏览器输入，构造Http数据
    + tomcat对数据进行解析，需要实现Http
    + tomcat从socket中区数据
4. socket时linux暴露的接口，时tcp_connect的对外接口
5. jAVA中 Socket是建立tcp连接，DatagramSocket是建立udp连接
    - socket0的方法是实现创建TCP中的native方法


[鲁班学院详解](https://www.bilibili.com/video/BV19E411j7cD?from=search&seid=6660309239296162043)

# 4 tomcat与JVM的关系
1. 一个tomcat是一个进程，其中有很多线程（与有多少个app无关）
2. 一个JVM启动一个tomcat，其中可以有很多APP
3. 一个tomcat中部署多个app，虽然同处一个JVM中，但是由于无法相互调用，所以也可以认为是分布式的
    - 一个tomcat只启动一个JVM，也就是说3个应用都是跑在一个JVM里，之所以它们不能互相调用是因为被类加载器隔离开的。 
    - 可见性规则：类只能看到由其类加载器的委托加载的其他类，委托是类的加载器及其所有父类加载器的递归集