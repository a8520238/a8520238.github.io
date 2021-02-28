---
title: Http
date: 2021-2-28
cover:
top_img:
categories: 计算机网络
tags: 
mathjax: true
katex: true
---
# Http1.0和Http1.1

1. http1.0
- 客户端可以与web服务器连接后，只能获得一个web资源，断开连接
- 浏览器阻塞：浏览器对于同一个域名，同时只能有4个连接
2. http1.1
- 缓存处理：在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。
- 带宽优化及网络连接的使用：HTTP1.0中，存在浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来，HTTP1.1在请求头引入range头域，它允许只请求资源的某个部分，即返回码是206.
- 错误通知管理：在HTTP1.1中新增了24个错误状态响应码。
- Host头处理：在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名。
- 长连接

3. HTTPS
- HTTPS协议需要到CA申请证书，一般免费证书很少，需要交费。
- HTTP协议运行在TCP之上，所有传输的内容都是明文，HTTPS运行在SSL/TLS之上，SSL/TLS运行在TCP之上，所有传输的内容都经过加密的。
- HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
- HTTPS可以有效的防止运营商劫持，解决了防劫持的一个大问题。

4. spdy（http1.x改进）
- SPDY位于HTTP之下，TCP和SSL之上，这样可以轻松兼容老版本的HTTP协议(将HTTP1.x的内容封装成一种新的frame格式)，同时可以使用已有的SSL功能。
    1. 使用多路复用，（多个请求共用一个tcp），降低了延迟
    2. 允许对多个请求设置优先级
    3. header进行压缩
    4. 基于https

 
2. http2.0
    1. http2.0与spdy区别
    - HTTP2.0 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS
    - HTTP2.0 消息头的压缩算法采用 HPACK http://http2.github.io/http2-spec/compression.html，而非 SPDY 采用的 DEFLATE http://zh.wikipedia.org/wiki/DEFLATE
    2. 其余改进与spdy差别不大

