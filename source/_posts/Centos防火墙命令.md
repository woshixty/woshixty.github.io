---
title: Centos的一些命令和windows的一些命令
date: 2020-11-10 15:22:59
tags: [centos, windows]
categories: Linux
---

参考博客
[Centos 防火墙命令](https://blog.csdn.net/qq_32923745/article/details/78048822)

(由于自己在使用云服务器的命令的时候常常要百度，故做一点总结)

### centos7防火墙命令

#### 查看开放端口

```powershell
firewall-cmd --list-ports
```

#### 开启端口

```powershell
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

#### 查看端口占用情况

```bash
lsof -i tcp:8090
```

> 会显示以下信息

```powershell
COMMAND   PID USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
java    27581 root   19u  IPv6 136588143      0t0  TCP *:8090 (LISTEN)

# lsof输出各列信息的意义如下：
# COMMAND    进程的名称
# PID        进程的标识符
# USER       进程的所有者
# FD         文件描述符，应用程序通过文件描述符识别该文件，如cwd，txt
# TYPE       文件类型
# DEVICE     指定磁盘的名称
# SIZE/OFF   文件大小
# NODE       文件在磁盘上的标识
# NAME       打开文件的确切名称
```

#### 开启范围端口

> 开启8080 - 9090范围的端口

```powershell
firewall-cmd  --zone=public  --add-port=8080-9090/tcp --permanent
```

命令含义：

- –zone								#作用域
- –add-port=80/tcp 	      #添加端口，格式为：端口/通讯协议
- –permanent                     #永久生效，没有此参数重启后失效

#### 防火墙命令

> 重启firewall

```bash
firewall-cmd --reload
```

> 查看防火墙状态

```bash
firewall-cmd  --state
```

> 开启firewall

```bash
systemctl start firewalld.service
```

> 停止firewall

```bash
systemctl stop firewalld.service
```

> 禁止firewall开机启动

```bash
systemctl disable firewalld.service
```



### centos7进程管理

linux上进程有5种状态:
1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)

#### 查看进程

- ps命令查找与进程相关的PID号

```bash
ps 9827
```

> 显示如下信息

```bash
  PID TTY      STAT   TIME COMMAND
 9827 pts/0    Sl     0:14 java -jar competitionrow-0.0.1-SNAPSHOT.jar
```



### centos7查看历史命令

- 查看历史命令

```shell
history
```

- 搜索历史命令

```shell
history | grep 'tmux'
# 模糊搜索tmux
```

出现以下结果：

```shell
  171  tmux new -s volunteer
  186  tmux ls
```





- 显示现行终端机下的所有程序，包括其他用户的程序

```bash
ps a
```

> 显示所有进程

```bash
  PID TTY      STAT   TIME COMMAND
 1321 tty1     Ss+    0:00 /sbin/agetty --noclear tty1 linux
 1322 ttyS0    Ss+    0:00 /sbin/agetty --keep-baud 115200 38400 9600 ttyS0 vt220
 9827 pts/0    Sl     0:14 java -jar competitionrow-0.0.1-SNAPSHOT.jar
17061 pts/0    R+     0:00 ps a
22278 pts/0    Ss     0:00 -bash
......
```



- 显示所有程序

```bash
ps -A
```

> 与上一个命令类似

```bash
  PID TTY          TIME CMD
    1 ?        00:12:32 systemd
    2 ?        00:00:02 kthreadd
    3 ?        00:01:53 ksoftirqd/0
    5 ?        00:00:00 kworker/0:0H
    7 ?        00:00:00 migration/0
    ......  
    #此处省略100行
```



- 查找特定的进程（如查找java进程）

```bash
ps aux | grep java
```

> 最常用的方法是ps aux,然后再通过管道使用grep命令过滤查找特定的进程,然后再对特定的进程进行操作

```bash
root      9827  0.5  8.8 2546924 166932 pts/0  Sl   14:03   0:14 java -jar competitionrow-0.0.1-SNAPSHOT.jar
root     18130  0.0  0.0 112708   976 pts/0    R+   14:48   0:00 grep --color=auto java
```



#### 杀死进程

- 通过进程id来杀

```bash
kill -9 pid
```

- 通过进程name来杀

```bash
killall -9 name
```



### windows端口进程管理

#### 查找端口

> 查找所有端口

```bash
netstat -ano
```

> 查找某个端口

```bash
netstat -aon|findstr "4000"
```

#### 杀死使用某个端口的进程

> 通过pid号

```bash
taskkill /f /pid  21972
```

> 成功杀死会有以下提示

成功: 已终止 PID 为 21972 的进程