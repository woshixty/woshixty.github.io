---
title: nginx安装以及入门
date: 2020-11-09 17:22:59
tags: [nginx, centos]
categories: tools
---
参考笔记:

[CentOS 7 下 yum 安装和配置 Nginx](https://blog.csdn.net/weixin_38118016/article/details/89949131)

[Nginx 入门指南](https://juejin.im/post/6844904129987526663)



> nginx如何安装

### yum安装nginx

1.**首先查看 Linux distribution 的版本号**

```shell
[root@VM_0_4_centos ~]# cat /etc/redhat-release

CentOS Linux release 7.5.1804 (Core) 
```



2.**Nginx 不在默认的 yum 源中，首先将Nginx加入到 yum 源里**

```powershell
[root@VM_0_4_centos ~]# rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

获取http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
警告：/var/tmp/rpm-tmp.a7nthc: 头V4 RSA/SHA1 Signature, 密钥 ID 7bd9bf62: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:nginx-release-centos-7-0.el7.ngx ################################# [100%]
```



3.**然后执行 yum repolist 查看一下，发现 Nginx 已经安装到本机了**

```powershell
[root@VM_0_4_centos ~]# yum repolist

已加载插件：fastestmirror, langpacks
Determining fastest mirrors
epel                                                     | 4.7 kB     00:00     
extras                                                   | 2.9 kB     00:00     
nginx                                                    | 2.9 kB     00:00     
os                                                       | 3.6 kB     00:00     
updates                                                  | 2.9 kB     00:00     
(1/3): epel/7/x86_64/updateinfo                            | 1.0 MB   00:00     
(2/3): epel/7/x86_64/primary_db                            | 6.9 MB   00:01     
(3/3): nginx/x86_64/primary_db                             |  56 kB   00:03     
源标识                      源名称                                        状态
epel/7/x86_64               EPEL for redhat/centos 7 - x86_64             13,455
extras/7/x86_64             Qcloud centos extras - x86_64                    413
nginx/x86_64                nginx repo                                       194
os/7/x86_64                 Qcloud centos os - x86_64                     10,070
updates/7/x86_64            Qcloud centos updates - x86_64                 1,134
repolist: 25,266
```



4.**安装nginx**(慢慢等待,需要一段时间)

```powershell
[root@VM_0_4_centos ~]# yum install nginx


#安装成功以后
已安装:
  nginx.x86_64 1:1.18.0-1.el7.ngx                                               

完毕！
```



> nginx简单配置

1.**查看 Nginx 版本**

```powershell
[root@VM_0_4_centos ~]# nginx -v

nginx version: nginx/1.18.0
```



2.**启动 Nginx 服务**

```powershell
[root@VM_0_4_centos ~]# systemctl start nginx
```



3.**访问 Nginx 服务**(本地访问一下，可以看到 Welcome to nginx)

```powershell
[root@VM_0_4_centos ~]# curl -i localhost

HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Tue, 27 Oct 2020 02:51:03 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 21 Apr 2020 15:07:31 GMT
Connection: keep-alive
ETag: "5e9f0c33-264"
Accept-Ranges: bytes

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



4.远程通过公网IP访问( ip:80 )

![nginx](https://cos-1301609895.cos.ap-nanjing.myqcloud.com/nginx.png)



### nginx源码包安装

[下载地址](https://nginx.org/en/download.html)

下载好之后，拷贝到/usr/local/nginx下

- 依赖库安装

```powershell
# 1.安装 gcc 环境
# nginx 编译时依赖 gcc 环境
[root@VM_0_4_centos nginx-1.18.0]# yum -y install gcc gcc-c++

# 2.安装 pcre
# 让 nginx 支持重写功能
[root@VM_0_4_centos nginx-1.18.0]# yum -y install pcre pcre-devel

# 3.安装 zlib
# zlib 库提供了很多压缩和解压缩的方式，nginx 使用 zlib 对 http 包内容进行 gzip 压缩
[root@VM_0_4_centos nginx-1.18.0]# yum -y install zlib zlib-devel

# 4.安装 openssl
# 安全套接字层密码库，用于通信加密
[root@VM_0_4_centos nginx-1.18.0]# yum -y install openssl openssl-devel

```

- 解压缩

```powershell
[root@VM_0_4_centos nginx]# tar -zxvf  nginx-1.18.0.tar.gz 
```

- 进入nginx-1.18.0目录下面进行源码编译安装

```powershell
[root@VM_0_4_centos nginx-1.18.0]# ./configure --prefix=/usr/local/nginx
# 检查平台安装环境
# --prefix=/usr/local/nginx  是 nginx 编译安装的目录（推荐），安装完后会在此目录下生成相关文件
# 如果前面的依赖库都安装成功后，执行 ./configure --prefix=/usr/local/nginx 命令会显示一些环境信息。如果出现错误，一般是依赖库没有安装完成，可按照错误提示信息进行所缺的依赖库安装。
```

- 进行源码编译并安装 nginx

```powershell
[root@VM_0_4_centos sbin]# make

[root@VM_0_4_centos sbin]# make install
```

- nginx服务操作命令

```powershell
#启动服务
[root@VM_0_4_centos sbin]# /usr/local/nginx/sbin/nginx

#重新加载服务
[root@VM_0_4_centos sbin]# /usr/local/nginx/sbin/nginx -s reload

#停止服务
[root@VM_0_4_centos sbin]# /usr/local/nginx/sbin/nginx -s stop

#查看nginx 服务进程
[root@VM_0_4_centos sbin]# ps -ef | grep nginx 
```

### nginx简单入门

>  什么是nginx?

Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。



> niginx有什么用?  (四大作用)

- 动静分离

动静分离其实就是 Nginx 服务器将接收到的请求分为动态请求和静态请求

静态请求直接从 nginx 服务器所设定的根目录路径去取对应的资源，动态请求转发给真实的后台去处理

这样做不仅能给应用服务器减轻压力，将后台api接口服务化，还能将前后端代码分开并行开发和部署

例如:

```powershell
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            root   html; # Nginx默认值
            index  index.html index.htm;
        }
        
        # 静态化配置，所有静态请求都转发给 nginx 处理，存放目录为 my-project
        location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
            root /usr/local/var/www/my-project; # 静态请求所代理到的根目录
        }
        
        # 动态请求匹配到path为'node'的就转发到8002端口处理
        location /node/ {  
            proxy_pass http://localhost:8002; # 充当服务代理
        }
}
```



- 反向代理

反向代理其实就类似你去找代购帮你买东西(浏览器或其他终端向nginx请求)，你不用管他去哪里买，只要他帮你买到你想要的东西就

行(浏览器或其他终端最终拿到了他想要的内容，但是具体从哪儿拿到的这个过程它并不知道)

配置反向代理:

```powershell
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            root   html; # Nginx默认值
            index  index.html index.htm;
        }
        
        proxy_pass http://localhost:8000; # 反向代理配置，请求会被转发到8000端口
}

```



- 负载均衡

随着业务的不断增长和用户的不断增多，一台服务已经满足不了系统要求了。这个时候就出现了服务器 [集群](https://www.cnblogs.com/bhlsheji/p/4026296.html)。

在服务器集群中，Nginx 可以将接收到的客户端请求“均匀地”（严格讲并不一定均匀，可以通过设置权重）分配到这个集群中所有的服务

器上。这个就叫做**负载均衡**。

配置负载均衡

```powershell
# 负载均衡：设置domain
upstream domain {
    server localhost:8000;
    server localhost:8001;
}
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            # root   html; # Nginx默认值
            # index  index.html index.htm;
            
            proxy_pass http://domain; # 负载均衡配置，请求会被平均分配到8000和8001端口
            proxy_set_header Host $host:$server_port;
        }
}
```



- 正向代理

正向代理跟反向道理正好相反。拿上文中的那个代购例子来讲，多个人找代购购买同一个商品，代购找到买这个的店后一次性给买了。这

个过程中，该店主是不知道代购是帮别代买买东西的。那么代购对于多个想买商品的顾客来讲，他就充当了正向代理。



### nginx删除

当有一天，你不再爱nginx时，你将如何删除它呢

- yum卸载nginx

```powershell
[root@VM_0_4_centos local]# yum remove nginx

Loaded plugins: fastestmirror, langpacks
Resolving Dependencies

......

Removed:
  nginx.x86_64 1:1.18.0-1.el7.ngx                                                          
Complete!
```

- 查看nginx是否还存在

```bash
[root@VM_0_4_centos ~]# find / -name nginx
[root@VM_0_4_centos ~]# 
```
