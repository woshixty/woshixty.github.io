### 镜像与容器的关系

镜像可以理解为一种构建时（build-in）结构，容器可以理解为一种运行时（run-time）结构

### 查看镜像

```shell
docker image ls
```

### 拉取镜像

```shell
docker image pull ubuntu:latest
```

### 镜像命名

- 镜像仓库服务

Docker镜像存储在镜像仓库服务（Image Registry）当中，Docker客户端的镜像仓库服务是可配置的，默认使用Docker Hub

镜像仓库服务包含多个镜像仓库（Image Repository）

一个镜像仓库包含多个镜像



- 镜像命名与标签

只需要给出镜像的名字与标签，就能够在官方的仓库中定位一个镜像，如下拉取镜像的命令：

> docker image pull <repository>:<tag>

```shell
# 从官方的Mongo库中拉取标签为3.3.11的镜像
docker image pull mongo:3.3.11
```



- 为镜像打多个标签

使用`-a`获取一个仓库中所有的镜像

例如下面的命令中就是拉取标签为`latest`的镜像

```shell
docker image pull ubuntu:latest
```



- 过滤`docker image ls`的输出内容

Docker提供`--filter`参数来过滤`docker image ls`命令返回的镜像列表内容



- 通过CLI方式搜索Docker Hub

`docker search`命令允许通过CLI的方式搜索`Docker Hub`，读者可以通过“NAME”字段的内容进行匹配，并且基于返回内容中任意列的值进行过滤

```shell
docker search nigelpoulton
```



- 镜像和分层

- 共享镜像层

- 根据摘要拉取镜像

- 