---
title: springcloudalibaba
date: 2021-2-28
cover:
top_img:
categories: 分布式
tags: 
mathjax: true
katex: true
---
# 1 nacos实时QPS

服务端内部保存滑动时间窗口，存储当前一秒QPS，会推送到客户端，由客户端进行服务限流降级等操作。客户端的操作行为也会被推送到服务端，从而形成QPS

# 2 组件
1. 服务发现组件nacos
2. 配置中心组件nacos
3. 断路保护组件sentinel
4. 远程组件dubbo,而原来是Openfeign
5. seata分布式事务