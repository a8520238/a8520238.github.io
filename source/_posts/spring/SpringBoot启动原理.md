---
title: springboot启动原理
date: 2021-2-28
cover:
top_img:
categories: springboot
tags: 
mathjax: true
katex: true
---
# SpringBoot原理

## 1 创建SpringApplicationContext

![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/06FF0675D7F041D8BFDED30D3D7E24FC/7943)

1. 创建SpringApplication对象
    ```
    //保存sources
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
    //判断是否是web
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //setInitializers
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
    //setlisteners
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    //检测哪个application中有main方法
    this.mainApplicationClass = this.deduceMainApplicationClass();
    ```
![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/C327D4D00F9943A98F38753D7DA11DB8/7958)
![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/118E03B71550445EB65F6E956EA7F56B/7961)
![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/BB45B6FFC11648518CB04DCB3116451F/7965)
![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/3176CB5FC72B46499BD282A8F73CD2AE/7967)

## 2 自定义starter

![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/EEEBD2377E4543359C92B2CE079201C1/7979)
![](http://note.youdao.com/yws/public/resource/397941bed8e986b1639771c6ec7e664d/xmlnote/8E8C9D49DDC842F8B44B4753EB234E92/7981)