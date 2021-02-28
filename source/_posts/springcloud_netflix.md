---
title: springcloud_netflix
date: 2021-2-28
cover:
top_img:
categories: 分布式
tags: 
mathjax: true
katex: true
---
# springcloud生态
[springcloud](https://www.bilibili.com/video/BV1jJ411S7xr?p=15)
## 1 4 个问题
1. API （解决网关）
2. HTTP， RPC（解决分布式通讯）
3. 注册和发现（解决高可用）
4. 熔断机制（进行服务降级）

微服务：将单一的应用程序划分成一组小的服务

> mycat 数据库读写分离  elastic search

spring cloud netflix

spring cloud 中国社区·

eureca是对应zookeeper的

## 2 Eureka
是Netflix的一个子模块，是一个机遇rest的服务
1. 导入依赖
2. 编写配置文件
3. 开启这个功能@Enable
4. 配置类

eureka好死不如赖活着

eureka与zookeeper区别

CAP三选二
C 强一致性
A 可用性
P 分区容错性

zookeeper保证的是CP，enreka保证的是AP

> 面试题，两者的区别

## 3 ribbon

1. spring cloud ribbon 是基于netflix实现的一套客户端负载均衡工具

nginx是集中时，由nginx决定分发给谁

ribbon是进程时，由消费方通过注册中心获取所有可用，通过算法决定如何选择。

> feign是面向接口版本的ribbon, 也是负载均衡的

## 4 Hystrix 

Hystrix 可以提供服务熔断和服务降级

服务熔断机制是对应雪崩效应的一种微服务链路保护机制

相关概念，备份，服务降级

服务熔断是提供者，服务降级是消费者

降级是客户端的，因为后台为了释放资源已经关闭了，可以用feign配合hystrix来进行

服务熔断：服务端，某个服务超时或者异常，引起熔断，保险丝

服务降级：客户端，从整体网站请求负载考虑，当某个服务熔断或者关闭之后，服务将不再会被调用，不走服务器。此时在客户端，我们可以准备一个FallbackFactory，返回一个默认的值，但整体服务水平下降了。

# 5 路由网关 zuul
1. 身份验证和安全性
2. 监控
3. 动态路由
4. 压力测试
5. 减载
6. 静态响应处理
7. 多区域弹性

> 统一路由，统一权限认证

通过配置文件对请求进行路由和过滤

# 6 spring cloud config

config为微服务架构中的微服务提供集中化的外部配置支持 


