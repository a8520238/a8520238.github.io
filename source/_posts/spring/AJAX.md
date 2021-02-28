---
title: ajax
date: 2021-2-28
cover:
top_img:
categories: springmvc
tags: 
mathjax: true
katex: true
---
# AJAX（Asynchronous JavaScript and XML）

## 1 AJAX原理
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/85F8488612E24361A05A0AAB2644380F/7894)
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/8CAB11502B634CF589949A8E15CCED2B/7905)
也就是说后端不再用forward()或者redirect()进行转发或重定向，而是通过返回callback()函数，(在前段可视作为success()或error())，这样视图部分由前端完成，后台只负责传递请求中的数据@RestController。

> 取当前项目的绝对路径
${pageContext.request.contextPath}

Todo:
- 函数：闭包() ()
- Dom
    + id, name, tag
    + creat, remove
- Bom
    + window
    + document
- ES6

