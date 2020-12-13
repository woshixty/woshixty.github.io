---
title: Redis的安装与简单使用
date: 2020-12-05
tags: [centos, redis]
categories: tools
---



### Redis其他配置以及集群

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

