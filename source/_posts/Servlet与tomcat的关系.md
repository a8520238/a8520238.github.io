---
title: servlet和tomcat的关系
date: 2021-2-28
cover:
top_img:
categories: javaEE
tags: 
mathjax: true
katex: true
---
# Servlet与Servlet容器的关系

> Servlet是用Java编写的Server端程序，它与协议和平台无关。

![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/5A43686456B743739451E4CF0DA9AD01/6836)

> 接口是连接Servlet与Servlet容器的关键。
## 1 Context容器

Tomcat 的容器等级中，Context 容器是直接管理 Servlet 在容器中的包装类 Wrapper，所以 Context 容器如何运行将直接影响 Servlet 的工作方式。

真正管理 Servlet 的容器是 Context 容器，一个 Context 对应一个 Web 工程，在 Tomcat 的配置文件中可以很容易发现这一点，如下：

```
<Context path="/projectOne " docBase="D:\projects\projectOne"
reloadable="true" />
```
## 2 Servlet容器的启动过程

Tomcat7 也开始支持嵌入式功能，增加了一个【启动类 org.apache.catalina.startup.Tomcat】。创建一个实例对象并调用 start 方法就可以很容易启动 Tomcat，我们还可以通过这个对象来增加和修改 Tomcat 的配置参数，如可以动态增加 Context、Servlet 等。我们就选择 Tomcat7 自带的 examples Web 工程，并看看它是如何加到这个 Context 容器中的。

```
清单 1:给 Tomcat 增加一个 Web 工程:
Tomcat tomcat = getTomcatInstance(); 
File appDir = new File(getBuildDirectory(), "webapps/examples"); 
tomcat.addWebapp(null, "/examples", appDir.getAbsolutePath()); 
tomcat.start(); 
ByteChunk res = getUrl("http://localhost:" + getPort() + 
              "/examples/servlets/servlet/HelloWorldExample"); 
assertTrue(res.toString().indexOf("<h1>Hello World!</h1>") > 0);
```
清单 1的代码是创建一个 Tomcat 实例并新增一个 Web 应用，然后启动 Tomcat 并调用其中的一个 HelloWorldExample Servlet，看有没有正确返回预期的数据。
```
清单 2:Tomcat 的 addWebapp 方法的代码如下：
public Context addWebapp(Host host, String url, String path) { 
       silence(url); 
       Context ctx = new StandardContext(); 
       ctx.setPath( url ); 
       ctx.setDocBase(path); 
       if (defaultRealm == null) { 
           initSimpleAuth(); 
       } 
       ctx.setRealm(defaultRealm); 
       ctx.addLifecycleListener(new DefaultWebXmlListener()); 
       ContextConfig ctxCfg = new ContextConfig(); 
       ctx.addLifecycleListener(ctxCfg); 
       ctxCfg.setDefaultWebXml("org/apache/catalin/startup/NO_DEFAULT_XML"); 
       if (host == null) { 
           getHost().addChild(ctx); 
       } else { 
           host.addChild(ctx); 
       } 
       return ctx; 
}
```
- 添加一个 Web 应用时将会创建一个 StandardContext 容器，并且给这个 Context 容器设置必要的参数:
    + url 代表这个应用在 Tomcat 中的访问路径；
    + path 代表这个应用实际的物理路径）
- 最重要的一个配置是 ContextConfig，【ContextConfig监听器】继承了 【LifecycleListener 监听器接口】，它是在调用清单 2 时被加入到 StandardContext 容器中。
- 当 Context 容器初始化状态设为 init 时，添加在 Context 容器的 Listener 将会被调用。【ContextConfig监听器】将会负责整个 Web 应用配置文件的解析工作。
最后将这个 Context 容器加到父容器 Host 中。

## 3 【ContextConfig监听器】的解析工作有哪些

1. ContextConfig 的 init 方法将会主要完成以下工作：
    - 创建用于解析 xml 配置文件的 contextDigester 对象
    - 读取默认 context.xml 配置文件，如果存在解析它
    - 读取默认 Host 配置文件，如果存在解析它
    - 读取默认 Context 自身的配置文件，如果存在解析它
    - 设置 Context 的 DocBase

2. ContextConfig 的 init 方法完成后，Context 容器的会执行 startInternal 方法，这个方法启动逻辑比较复杂，主要包括如下几个部分：
    - 创建读取资源文件的对象
    - 创建 ClassLoader 对象
    - 设置应用的工作目录
    - 启动相关的辅助类如：logger、realm、resources 等
    - 修改启动状态，通知感兴趣的观察者（Web 应用的配置）
    - 子容器的初始化
    - 获取 ServletContext 并设置必要的参数
    - 初始化“load on startup”的 Servlet

[servlet和tomcat关系](https://blog.csdn.net/baidu_36583119/article/details/79642407)