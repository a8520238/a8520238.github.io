---
title: docker
date: 2021-2-28
cover:
top_img:
categories: 项目相关
tags: 
mathjax: true
katex: true
---
资源[Docker](https://www.bilibili.com/video/BV1og4y1q7M4/?spm_id_from=333.788.videocard.0)

[docker基础](https://www.bilibili.com/video/BV145411n7Ab?from=search&seid=6385530235335936447)
# 1 虚拟，docker可以理解为容器
1. docker能保证一直的运行环境
2. 数组内核，不需要硬件虚拟以及运行完整操作系统等额外开销，对资源利用效率高
3. 不需要vmware那样启动其他操作系统，直接运行与宿主内核，启动快。
4. 可以保证持续交付和部署，一次创建到处运行，迁移轻松

# 2 docker和vm区别
1. vm: host os -> hypervisor -> guest os -> bin
2. docker: host os -> docker engine -> bins
> docker没有guest os，是基于基层操作系统，不需要安装额外的操作系统

# 3 Docker组件
1. docker客户端通过访问docker主机的守护进程访问docker容器

# 4 启动命令
1. systemctl start docker启动
2. systemctl status docker查看状态
3. docker info 查看状态
4. systemctl stop docker停止
5. systemctl enable docker重新启动 

# 5 镜像命令
1. docker images 查看镜像
    - 这些景象都是存储在Docker宿主机的/var/lib/docker目录下
2. docker search 搜索镜像
3. docker pull 镜像名称 拉去镜像
4. docker rmi 镜像ID
5. docker rmi `docker images -q`删除所有镜像

# 6 容器命令
## 6.1 查看
1. docker ps 查看容器
2. docker ps -a 查看所有容器
3. docker ps -l 查看最后一次运行的容器
4. docker ps -f status=exited 查看停止的容器
## 6.2 运行
1. docker run
    - \- i 表示运行容器
    - \- t  表示容器启动后进入命令行模式
    - \- v 表示目录映射关系（前者宿主机目录，后者是映射到宿主机上的目录）
    - \- p 端口映射
    - \- d 创建后台守护容器
2. 交互式创建
    - docker run -it --name=mycentos centos /bin/bash
3. 后台创建
    - docker run -di --name=mycentos2 centos
    - docker exec -it mycentos2 /bin/bash
4. 停止后台
    - 普通的exit就可
    - 后台 docker stop id
    - 后台重启 docker start id
## 6.3 拷贝
1. 宿主机拷贝到容器：docker cp 文件 容器id:/usr/local
2. docker exec -it id /bin/bash 启动后发现拷贝成功
3. 容器拷贝到宿主机：docker cp mycentos2:/usr/local/文件 文件

## 6.4 目录挂载
1. 创建目录 mkdir -p /usr/local/mydata
2. 创建文件 vim /usr/local/mydata
3. 创建容器 docker run -di -v /usr/local/mydata/:/usr/local/mydata --name=mycentos3 centos
4. 运行 docker exec -it mycentos3 /bin/bash
5. 可以看到创建的文件
> 上述是挂载，也叫映射

- docker inspect --format='{{.NetworkSettings.IPAddress}}' mycentos3
- docker rm 删除容器

## 6.5 docker部署mysql
1. docker pull mysql
2. docker run -di --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
3. docker exec -it mysql /bin/bash
4. mysql -uroot -proot --default-character-set=utf8

> linux查看端口号 netstat -tanlp

## 6.6 docker 创建nginx
1. docker pull nginx
2. docker run -di --name=nginx -p 80:80 nginx
3. docker cp nginx:/etc/nginx /usr/local/mydata/nginx/
4. 准备挂在 docker stop nginx 
5. docker rm nginx
6. docker run -di --name=nginx -p 80:80 -v /usr/local/mydata/conf/:/etc/nginx nginx

## 6.7 docker 创建redis
1. docker pull redis
2. docker run -di --name=redis -p 6379:6379 redis

## 6.8 docker创建rabbitmq
1. docker pull rabbitmq
2. docker run -di --name=rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq
3. docker exec -it rabbitmq /bin/bash
4. rabbitmq-plugins enable rabbitmq_management

## 6.9 elasticsearch部署

1. docker pull elasticsearch:7.5.0
2. 修改虚拟内存大小 sysctl -w vm.max_map_count=262144
3. docker run -di --name=elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "cluster.name=elasticsearch" -v /usr/local/mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins elasticsearch:7.5.0
4. docker exec -it elasticsearch /bin/bash
5. elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.5.0/elasticsearch-analysis-ik-7.5.0.zip  安装中文分词器
6. docker restart elasticsearch
> 通过9200访问

## 6.10 zookeeper部署
1. docker pull zookeeper:3.4.13
2. docker run -di --name=zookeeper -p 2181:2181 zookeeper:3.4.13

# 7 容器的备份与迁移
 
 ## 7.1 容器打包成镜像
 1. docker commit redis myredis 保存
 2. docker run -di --name=myredis myredis 重新创建
 
## 7.2 备份
1. docker save -o myredis.tar myredis 打包成tar包
2. docker load -i myredis.tar 恢复镜像

# 8 Dockerfile

以centos jdk8为例，在宿主环境下运行
1. mkdir -p /usr/local/dockerjdk8 先创建工作目录
2. 将jdk的tar包移入当前目录
3. vim Dockerfile 开启编写
4. FROM centos:7
5. MAINTAINER xxxx （创建者信息）
6. WORKDIR /usr
7. RUN mkdir /usr/local/java (创建目录)
8. ADD jdk-8u202-linux-x64.tar.gz /usr/local/java（将jdk8的 tar包拷贝到）
9. ENV JAVA_HOME /usr/local/java/jdk1.8.0_202
10. ENV PATH $JAVA_HOME/bin:$PATH

> entrypoint首先执行的命令 CMD在其之后执行的

准备好Dockerfile和jdk tar包后，在该目录下
1. docker build -t='jdk1.8' .

# 9 Docker 私有仓库

1. docker pull registry 拉取私有仓库镜像
2. docker run -di --name=registry -p 5000:5000 registry
3. 访问ip:5000/v2/_catalog
4. vim /etc/docker/daemon.json, 在其中添加新人配置 "insecure-registries":["ip:5000"]
5. systemctl restart docker
6. docker start registry

将镜像上传私有仓库
1. docker jdk1.8 ip:5000/jdk1.8
2. docker push ip:5000/jdk1.8
3. 使用的时候先添加信任，然后直接poll

# 10 DockerMaven插件
1. vim /lib/systemd/system/docker.service
2. ExecStart改为
```
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375
```
3. systemctl daemon-reload
4. systemctl restart docker
5. docker start registry
6. docker-maven-plugins

# 11 重启错误

# rm -rf /var/lib/docker/
# 添加如下内容
# vim /etc/docker/daemon.json
{
      "graph": "/mnt/docker-data",
      "storage-driver": "overlay"
} 

# 12 Docker Compose
Docker Compose  定义运行多个容器
1. 需要Dockerfile 
2. docker-compose.yml
3. run docker-compose up

- Compose是Docker官方的开源醒目

阶段
1. docker镜像 run => 容器
2. Dockerfile 构建镜像（服务打包）
3. docker-compose启动项目（编排多个微服务环境）
4. Docker网络

## 12.1 yaml规则

```
# 3层
version: '' #版本
services: #服务
    服务1: web
    # 配置
        images
        build 
        network
    服务2: redis
# 其他配置 网络/卷
volumns:
networks:
configs:
```

# 12.2 部署

1. 下载项目 （docker-compose.yaml）
2. 如果需要文件Dockerfile
3. 准备好一键启动 run docker-compose up   比如wordpress博客

前台启动： docker -d
后台启动： docker-compose up -d

docker-compose up --build

curl localhost (curl http获取连接)
# 13 Docker Swarm
集群方式部署

1. 工作模式 管理节点 + 工作节点

2. 集群配置 
    1. 主节点 docker swarm init --advertise-addr 私网地址
    2. docker swarm join 加入一个节点
        - 获取令牌 docker swarm join-token manager
        - 获取worker令牌 docker swarm join-token worker
        - docker swarm join --token SWMTKN..令牌 ip:端口 向主节点加入工作节点/主节点

3. Raft协议一致性算法
Raft:保证大多数节点存活才可以用，集群至少有三台，至少两个主节点
4. 体会
    - swarm将docker看成服务
    - 灰度发布：金丝雀发布（不停服务）
    - docker service create -p 8888:80 --name my-nginx nginx 
    - docker dun 容器启动 不具有扩缩容器
    - docker service 服务，具有扩缩容器
    - docker service ls服务
    - docker service update --relipcas 3 my-nginx 动态扩缩容
    - docker service update --relipcas 3 my-nginx 回滚到一个节点
    - docker service scale my-nginx=5 也是动态扩缩容 和上两个update命令一样
    - 搭建集群，启动服务，动态管理容器
5. 概念
    - swarm 集群管理和编号，docker可以初始化一个swarm集群，其他节点可以加入
    - node 就是一个docker节点，多个节点组成一个网络集群（管理，工作）
    - service，可以在管理节点或者工作节点运行。用户访问的地方，多个task的集合，用户不需要关心，swarm实现。
    - task，容器内的命令，细节任务
    - 命令 -> 管理 -> api -> 调度 -> 工作节点（创建task容器维护创建）

6. 网络模式
    1. overlay 所有的机器共用一个网络
    2. ingress: 特殊的overlay网络，负载均衡的功能