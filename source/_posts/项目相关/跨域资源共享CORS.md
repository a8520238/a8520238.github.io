---
title: 跨域资源共享CORS
date: 2021-2-28
cover:
top_img:
categories: 项目相关
tags: 
mathjax: true
katex: true
---
# 1 对AJAX的理解

[跨域资源共享 CORS 详解](https://www.itzhai.com/articles/insight-into-the-underlying-architecture-of-mysql-buffer-and-disk.html)

[CORS后端代码](https://blog.csdn.net/weixin_30509393/article/details/95305932)

[对跨域的理解](https://segmentfault.com/a/1190000015597029)

可以这样理解，AJAX是需要那部分数据再向后台去取，避免了一开始全部加载，也避免了只需要更新一部分数据而重新请求整个页面。

# 2 跨域资源共享 CORS

## 2.1 XMLHttpRequest 对象
XMLHttpRequest 对象用于在后台与服务器交换数据。支持：
1. 在不重新加载页面的情况下更新网页
2. 在页面已加载后从服务器请求数据
3. 在页面已加载后从服务器接收数据
4. 在后台向服务器发送数据

> CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

> 它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制

## 1 简单请求
只要同时满足以下两大条件，就属于简单请求。
1. 请求方法是以下三种方法之一：
- HEAD
- GET
- POST
2. HTTP的头信息不超出以下几种字段：
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
1. 对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。
2. 浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段。
```
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
Origin字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

```
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```
上面的头信息之中，有三个与CORS请求相关的字段，都以Access-Control-开头。