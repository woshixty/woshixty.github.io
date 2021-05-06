---
title: Docker安装Nginx、Tomcat、Redis
date: 2020-12-05
tags: [centos, redis, nginx, tomcat, redis]
categories: Tools
---



### Nginx的安装与部署

#### 1.搜索nginx镜像

```shell
docker search nginx
```

#### 2.拉取下载nginx镜像 (直接去Docker搜索可以看到帮助文档)

```shell
docker pull nginx
```

#### 3.康康有没有拉取成功

```shell
docker images
```

>  显示有以下这一行则表示拉取成功： nginx         latest    ae2feff98a0c   3 weeks ago     133MB

#### 4.启动我们的镜像

```shell
docker run -d --name nginx01 -p 3344:80 nginx
```

> 里面的 “-p 3344:80” 的意思是将 外部的 3344 端口映射到 容器内的 80 端口

#### 5.查看刚刚启动的容器

```shell
docker ps
```

会显示以下提示信息：

```shell
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                               NAMES
985e1a5fe8c2   nginx       "/docker-entrypoint.…"   7 seconds ago   Up 6 seconds   0.0.0.0:3344->80/tcp                nginx01
```

测一测本机的 3344 端口，可以看到能访问到 nginx

```shell
[root@iz2ze42tl10pjoj8i0za7hz ~]# curl localhost:3344

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### 6.进入容器康康

```shell
docker exec -it nginx01 /bin/bash
```

进来容器之后我们找一找配置文件，可以进行修改

```shell
root@985e1a5fe8c2:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@985e1a5fe8c2:/# cd /etc/nginx/
root@985e1a5fe8c2:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf
```



### Tomcat的安装与部署





### Redis配置

> 修改 /opt/docker_redis/docker-compose.yml 文件，以方便后期修改 Redis 配置信息

```yml
version: '3.1'
services:
  redis:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis
    environment:
     - TZ=Asia/Shanghai
    ports:
     - 6379:6379
    #指定一个关于redis的配置文件，映射redis配置文件
    volumes:
     - ./conf/redis.conf:/usr/local/redis/redis.conf
     - ./data:/data #映射data目录
    #运行容器的时候加在它
    command: ["redis-server","/usr/local/redis/redis.conf"]
```

> redis的配置

```yaml
#redis的AUTH密码
requirepass xxxx

#RDB持久化
# save 配置代表 RDB 执行的时机
save 900 1    # 900 秒内有一个key改变就执行
save 300 10    # 同上
save 60 10000    # 同上

#开启RDB持久化的压缩
rdbcompression yes

#RDB持久化文件的名称
dbfilename dump.rdb

# 修改redis.conf 配置文件223行左右位置 daemonize默认为no、修改为yes即可守护进程模式后台启动
daemonize yes

# 密码默认为空、若要设置密码、可以通过789行位置放开注释# requirepass foobared
requirepass 密码
```

