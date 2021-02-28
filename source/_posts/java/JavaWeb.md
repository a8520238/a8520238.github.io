---
title: javaWeb
date: 2021-2-28
cover:
top_img:
categories: javaEE
tags: 
mathjax: true
katex: true
---
# JavaWeb

## 1 基本概念

- 静态web, 提供给所有人看的数据始终不会发生变化。
- 动态web，每个人在不同的时间不同的地点看到的信息都不相同。

### 1.1 默认端口合计
- tomcat: 8080
- mysql: 3306
- http:80
- https:443

## 2 浏览器与服务器交互的过程

假设我们配置虚拟主机www.gacl.cn,并绑定了8080端口，并配置了虚拟主机映射如下：
```
<Host name="www.gacl.cn" appBase="F:\JavaWebApps">
      
</Host>
```
客户机查找的过程如下：
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/D1C70AFCAB4B437EA3C064B1DFBDDFE4/6739)

当我们打开浏览器，在浏览器的地址栏中输入URL地址"http://www.gacl.cn:8080/JavaWebDemo1/1.jsp"去访问服务器上的1.jsp这个web资源的过程中，浏览器和服务器都做了神马操作呢，我们是怎么在浏览器里面看到1.jsp这个web资源里面的内容的呢？

浏览器和服务器做了以下几个操作：

　　1、浏览器根据主机名"www.gacl.cn"去操作系统的Hosts文件中查找主机名对应的IP地址。

　　2、浏览器如果在操作系统的Hosts文件中没有找到对应的IP地址，就去互联网上的DNS服务器上查找"www.gacl.cn"这台主机对应的IP地址。

　　3、浏览器查找到"www.gacl.cn"这台主机对应的IP地址后，就使用IP地址连接到Web服务器。**也就是通过IP定位到主机**

　　4、浏览器连接到web服务器后，就使用http协议向服务器发送请求，发送请求的过程中，浏览器会向Web服务器以Stream(流)的形式传输数据，告诉Web服务器要访问服务器里面的哪个Web应用下的Web资源。

5、浏览器做完上面4步工作后，就开始等待，等待Web服务器把自己想要访问的1.jsp这个Web资源传输给它。

　　6、服务器接收到浏览器传输的数据后，开始解析接收到的数据，服务器解析"GET /JavaWebDemo1/1.jsp HTTP/1.1"里面的内容时知道客户端浏览器要访问的是JavaWebDemo1应用里面的1.jsp这个Web资源，然后服务器就去读取1.jsp这个Web资源里面的内容，将读到的内容再以Stream(流)的形式传输给浏览器。
　　
## 3 三层架构

![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/3C07BADF838C4948A23549ADE5760FD3/7046)


# toDO
[jAVA WEB](https://www.cnblogs.com/xdp-gacl/p/3744053.html)