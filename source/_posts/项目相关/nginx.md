---
title: nginx
date: 2021-2-28
cover:
top_img:
categories: 项目相关
tags: 
mathjax: true
katex: true
---
# 1 nginx 安装

最好使用docker安装

[nginx详解视频](https://www.bilibili.com/video/BV1W54y1z7GM?p=13)
[nginx视频对应笔记](https://blog.csdn.net/m0_49558851/article/details/107786372)

nginx4种模式
1. 轮训
2. 指定权重 根据服务器性能
3. 一致性hash，每次访问固定
4. 响应时间短的优先分配