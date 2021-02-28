---
title: springmvc狂笔记
date: 2021-2-28
cover:
top_img:
categories: springmvc
tags: 
mathjax: true
katex: true
---
# SpringMVC

> SSM mybatic + Spring + SpringMVC , 重点是SpringMVC的执行流程

## 1 MVC(底层是servlet)

MVC是一种软件设计规范，model(dao, service) view controller(servlet(底层))。

- 命名习惯：
    + pojo(实体类)（User）
    + vo(视图类)(userVo)
- 回顾：JSP：本质就是一个Servlet

- 依赖记录
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/7A3F869688A4403CAF3EB4E697446E0F/6014)

## 2 dispatcherServlet

- MVC围绕DispatcherServlet，他表示前置控制器，会拦截会户发出的请求。
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/DBB40074C9374AFE862BF24BC661EA96/7693)

- 核心方法doService.
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/8664B40E79894EB49ADCEE91E8BB6F86/7723)


1. 用户访问页面
2. HandlerMapping为处理器映射，Dispatcher调用HandlerMapping，HandlerMapping根据请求url查找Handler。
3. HandlerExecution表示具体的Handler，其主要作用是根据url查找控制器，如在url查找控制器为hello
4. HandlerExecution将解析后的信息传递给DispatcherServlet，如解析控制器映射等。
5. HandlerAdapter表示处理适配器，其按照特定的规则去执行Handler.
6. Handler让具体的Controller执行。
7. Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView.
8. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。
9. DispatcherServlet调用视图解析器ViewResolver来解析HandlerAdapter传递的逻辑视图名。
10. 视图解析器将解析的逻辑视图名传给DispatcherServlet.
11. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图。
12. 最终视图呈现给用户。

相关配置
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/5CA99729CBD84CA68A0A8CBB85FD2CD1/7790)
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/FD38BAE9CC394F4EB7D5A6C3D2A28ADB/7796)
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/8DC5C105297246C6B14B811363526469/7794)

## 3 Restful风格

- 使用@pathVariable来读取@RequestMapping中的路径
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/F34C8E6285C84C01BA85659E2F35FEB8/7804)

- 非restful风格中，使用RequestParam
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/C8B68A407FE64C5ABAAA16E88EAAD052/7832)
## 4 转发

- url不变

![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/531DC42543F44038A596E7108DC59C07/7815)

## 5 重定向

![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/626FC77FE5AD46E2821D320E4388EC30/7823)

## 6 提交一个对象
- 只要前端提交的参数与Model对象中的吻合，即可

![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/8B44EAF6AF04454AA11AB7803A474D88/7839)

## 7 乱码解决

在MVC中要在xml中配置CharacterEncodingFilter

[乱码问题解决](https://www.bilibili.com/video/BV1aE41167Tu?p=13)

## 8 json

- Jackson
- fastJson

@RequestMapping中produces要配置
```
produces = "application/json;charset=utf-8"
或者anotataion-driven标签中配置
```

SimpleDateFormat处理Date


applicationContext.xml配置

![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/1D1B8D9D10F14CB281C942372C2A2D27/7886)

## 9 拦截器

- 过滤器(filter)和拦截器的区别
    + 拦截器是AOP思想
    + 拦截器只会拦截访问控制器的方法，如js等静态资源不会拦截
    + 拦截器是框架实现的，如mvc就有mvc的拦截器
- 过滤器需要实现HandlerIntecepter
![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/DB4DA61B6F27423C912178DC790FBA4F/7927)

## 10 文件上传与下载（Todo）