---
title: servlet
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
# Servlet相关知识整理

## 1 Servlet

a、servlet是运行在Web服务器中的小型Java程序，可以用过HTTP接受和响应Web客户端的请求。
b、servlet是实现了servlet接口的程序

```
public class ServletDemo implements Servlet {
    //重写方法service() 
    public void service(ServletRequest arg0, ServletResponse arg1) throws ServletException, IOException {
        System.out.println(""ok);
    }
}
```

![](http://note.youdao.com/yws/public/resource/5cf00375bd027272dff28d8755142c56/xmlnote/76CADB309CDC4214AED6EF8BF01356BE/5494)

## 2 Servlet执行过程

![](http://note.youdao.com/yws/public/resource/5cf00375bd027272dff28d8755142c56/xmlnote/4272F4CE99DB46899EF70523FFFB6BC4/5501)

## 3 servlet 生命周期

出生：实例化--->初始化 第一次访问(也可以设置在服务器启动时就创建)  init()
运行 service()
销毁：卸载应用 destory()

## 4 Servlet的三种创建方式

1. 实现javax.servlet.Servlet接口
2. 继承javax.servlet.GenericServlet类（适配器模式）
3. 继承javax.servlet.http.HttpServlet类（模板方法设计模式）实际中使用。模板指的是service方法，
有doGet和doPost两个方法。

> 正则匹配时，/优先于后缀名

## 5 GenericServlet与HttpServlet区别

1. HttpServlet

1). 是一个 Servlet, 继承自 GenericServlet. 针对于 HTTP 协议所定制.

2). 在 service() 方法中直接把 ServletReuqest 和 ServletResponse 转为 HttpServletRequest 和 HttpServletResponse.并调用了重载的 service(HttpServletRequest, HttpServletResponse)

在 service(HttpServletRequest, HttpServletResponse) 获取了请求方式: request.getMethod(). 根据请求方式有创建了
doXxx() 方法(xxx 为具体的请求方式, 比如 doGet, doPost)

3). 实际开发中, 直接继承 HttpServlet, 并根据请求方式复写 doXxx() 方法即可.

4). 好处: 直接由针对性的覆盖 doXxx() 方法; 直接使用 HttpServletRequest 和 HttpServletResponse, 不再需要强转.

2. GenericServlet

1). 是一个 Serlvet. 是 Servlet 接口和 ServletConfig 接口的实现类. 但是一个抽象类. 其中的 service 方法为抽象方法

2). 如果新建的 Servlet 程序直接继承 GenericSerlvet 会使开发更简洁.

3). 具体实现:

①. 在 GenericServlet 中声明了一个 SerlvetConfig 类型的成员变量, 在 init(ServletConfig) 方法中对其进行了初始化 
②. 利用 servletConfig 成员变量的方法实现了 ServletConfig 接口的方法
③. 还定义了一个 init() 方法, 在 init(SerlvetConfig) 方法中对其进行调用, 子类可以直接覆盖 init() 在其中实现对 Servlet 的初始化. 
④. 不建议直接覆盖 init(ServletConfig), 因为如果忘记编写 super.init(config); 而还是用了 SerlvetConfig 接口的方法,
则会出现空指针异常. 
⑤. 新建的 init(){} 并非 Serlvet 的生命周期方法. 而 init(ServletConfig) 是生命周期相关的方法.

如果为了开发一个能用在网页上服务于使用HTTP协议请求的Servlet，你的Servlet必须要继承自HttpServlet


## 6 servlet线程安全

- 如果service()方法没有访问Servlet的成员变量也没有访问全局的资源比如静态变量、文件、数据库连接等，而是只使用了当前线程自己的资源，比如非指向全局资源的临时变量、request和response对象等。该方法本身就是线程安全的，不必进行任何的同步控制。
- 如果service()方法访问了Servlet的成员变量，但是对该变量的操作是只读操作，该方法本身就是线程安全的，不必进行任何的同步控制。
- 如果service()方法访问了Servlet的成员变量，并且对该变量的操作既有读又有写，通常需要加上同步控制语句。
- 如果service()方法访问了全局的静态变量，如果同一时刻系统中也可能有其它线程访问该静态变量，如果既有读也有写的操作，通常需要加上同步控制语句。
- 如果service()方法访问了全局的资源，比如文件、数据库连接等，通常需要加上同步控制语句。


## 7 servletConfig与servletContext

[servletConfig与servletContext](https://blog.csdn.net/durenniu/article/details/81066817)

- getInitParameter() 获得初始化参数
- getServletConfig() 获得Servlet配置
- getServletContext() 获得Servlet上下文参数，代表该网站的参数

ServletContext
- 共享数据。多个Servlet参数共享


## 8 Servlet运行原理

- Servlet是由Web服务器调用，web服务器在收到浏览器请求之后，会
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/95A117DD816C4244B15DD63A7D23183A/6877)
 
- 配置文件中的Servlet路由原理，map
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/ADFEDD871BD34BE597A0CDD331DCBDC8/6893)

- dispatcher请求转发，不会重定向，只是请求转发，返回200
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/5445ED917B4546A095E4330B980B0D3A/6897)

- proporties读取配置,使用字节流
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/E7F383D83E5C470095100B84ACBF8085/6908)

## 9 HttpServletResponse

- 下载文件
    + 获取下载文件的路径
    + 下载的文件名
    + 设置想办法让浏览器能够支持下载我们需要的东西
    + 获取下载文件的输入流
    + 创建缓冲区
    + 获取OutputStream对象
    + 将FileOutputStream流写入到buffer缓冲区
    + 使用OutputStream将缓冲区中的数据输出到客户端
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/AF712385EE70496C830E01E1A1D6FF45/6936)

- 验证码
    + 前端实现
    + 后端实现，需要用到Java的图片类，生成一个图片
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/2018038E3B4D4756B8CF8BFD5CE09A89/6948)
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/0CADECC58A254D6E992D2F6585390CE5/6950)

- 实现重定向
    + 一个web资源B受到客户端A请求后，B他会通知A客户端去访问另外一个web资源C，这个过程叫重定向
    + 常见的应用场景：用户登录
    ```
    void sendRedirect(String var1) throws IOException;
    ```
- 重定向和转发的区别
    + 相同点页面都会跳转
    + 不同点，请求转发url不变，重定向会变

## 10 HttpServletRequest

客户端你的请求，HTTP请求中的所有信息被封装到这里。重点方法：
```
getParameter(String s) -> String
getParameterValues(String s) -> String[]
```

后台接受中文乱码问题解决：
```
req.setCharacterEncoding("utf-8");
resp.setCharacterEncoding("utf-8");
```
