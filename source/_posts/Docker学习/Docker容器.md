---
title: Docker容器
date: 2022-01-09
tags: [容器]
categories: Docker
---

### 一、简介

容器是镜像的运行时实例



### 二、详解

#### 1、启动一个简单的容器

启动容器的一个简单方式是：`docker container run`

启动一个简单的容器化版本的`Ubuntu Linux`

> 命令的基础格式为：`docker container run <options> <im- age>:<tag> <app>`

```shell
docker container run -it ubuntu:latest /bin/bash
```

- `-it`表示使容器具备交互性并于终端进行连接
- `/bin/bash`表示了运行在容器中的程序

有些指令无法执行，大部分容器镜像都是经过高度优化的，代表某些命令或者包没有安装



#### 2、容器进程



#### 3、容器生命周期



#### 4、优雅地停止容器

先`docker container stop`容器，再`docker container rm`容器



#### 5、利用重启策略进行容器的自我修复

- always
- unless-stopped
- on-failed



#### 6、web服务器示例

在8080端口启动一个简单的web服务

```shell
docker container run -d --name webserver -p 80:8080 \
nigelpoulton/pluralsight-docker-ci
```

- `-d`表示后台模式启动
- `-p 80:8080`表示将Docker主机的端口映射到容器内



#### 7、查看容器详情

在上个例子中，我们并没有指定容器中具体的应用，但是容器却运行了一个简单的web服务

当构建镜像时，可以通过嵌入指令来列出希望容器运行时启动的默认应用

如下：

```shell
docker image inspect nigelpoulton/pluralsight-docker-ci
```





