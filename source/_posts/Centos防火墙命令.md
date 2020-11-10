---
title: Centos的一些命令
date: 2020-11-10 15:22:59
tags: [centos]
categories: linux
---

参考博客
[Centos 防火墙命令](https://blog.csdn.net/qq_32923745/article/details/78048822)

(由于自己在使用云服务器的命令的时候常常要百度，故做一点总结)

### centos防火墙的一些命令

> #### **查看开放端口**

```powershell
firewall-cmd --list-ports
```

> #### **开启端口**

```powershell
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

> #### **开启范围端口 8080 - 9090范围的端口**

```powershell
firewall-cmd  --zone=public  --add-port=8080-9090/tcp --permanent
```

> #### **命令含义**

```
–zone                          #作用域
–add-port=80/tcp               #添加端口，格式为：端口/通讯协议
–permanent                     #永久生效，没有此参数重启后失效
```

> #### **防火墙命令**

```powershell
firewall-cmd --reload                  #重启firewall
firewall-cmd  --state                  #查看防火墙状态
systemctl start firewalld.service      #开启firewall
systemctl stop firewalld.service       #停止firewall
systemctl disable firewalld.service    #禁止firewall开机启动
```