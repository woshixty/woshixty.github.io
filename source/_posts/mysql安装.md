---
title: mysql的安装
date: 2020-11-11
tags: [centos, mysql]
categories: Tools
---

(我有个朋友，他服务器上的MySQL出问题了，直接重装，解决一切问题)

### 一、MySQL卸载

#### 1. yum康康安装的MySQL

> 使用 yum list installed | grep mysql 全都yum remove掉就可以了

```bash
yum list installed | grep mysql
```

最后显示`Complete!`就算成功了



#### 2. 赶尽杀绝

> 然后使用 find / -name mysql 依次将文件删除即可

```bash
find / -name mysql
```

将显示的文件统统删掉即可

- /var/lib/mysql

- /var/lib/mysql/mysql

- /usr/share/mysql

```bash
rm -rf /var/lib/mysql /var/lib/mysql/mysql /usr/share/mysql
```



### 二、rpm安装MySql

#### 1. 下载安装包

> 先去[官网](https://downloads.mysql.com/archives/community/)下载安装包，我这边是centos操作系统，所以选择Red Hat那个，我下载的是mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar 然后上传到服务器上

```bash
ls
```

康康安装包在不在：`mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar`



#### 2. 删除系统自带的东西

> 如果是第一次安装，检查一下系统自带的mariadb-lib以及MySQL 我这边干干净净呢

```bash
rpm -qa|grep mariadb
```

```bash
rpm -qa|grep mysql
```

如果显示有东西的话就用下面的命令弄掉它

```shell
rpm -e --nodeps + 搜索出来的文件名
```



#### 3. 安装依赖

> 依次安装下面三个依赖

```bash
yum install libaio
```

```bash
yum install perl
```

```bash
 yum install net-tools
```



#### 4.解压缩

> 先创建一个目录，然后在目录里 进行解压tar -xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar，并赋予目录最大权限

创建目录

```bash
mkdir mysql
```

进入目录

```bash
cd mysql/
```

解压

```bash
tar -xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
```

出去给目录赋予权限

```bash
cd ..
```

```bash
chmod -R 777 mysql
```



#### 5. 安装MySQL

> 严格按照下面四个步骤安装软件包 

```bash
rpm -ivh mysql-community-common-5.7.29-1.el7.x86_64.rpm
```

```bash
rpm -ivh mysql-community-libs-5.7.29-1.el7.x86_64.rpm 
```

```bash
rpm -ivh mysql-community-client-5.7.29-1.el7.x86_64.rpm 
```

```bash
rpm -ivh mysql-community-server-5.7.29-1.el7.x86_64.rpm 
```



#### 6. 配置数据库

> 修改配置文件

```bash
vim /etc/my.cnf
```

```sh
#在[mysqld]下面添加这三行
skip-grant-tables  #跳过登录验证
character_set_server=utf8  #设置默认字符集UTF-8
init_connect='SET NAMES utf8'  #设置默认字符集UTF-8
```



#### 7. 启动MySQL服务

> 执行下面两个命令，不要输入命令直接回车就好啦

```bash
systemctl start mysqld.service
```

```bash
mysql -uroot -p
```

这样就直接进入MySQL啦！



#### 8. 设置密码

> 在MySQL中执行下面的命令，设置密码并且让他生效

进入MySQL

```bash
mysql -uroot -p
```

更新密码，这只是暂时的密码，下次进入得修改

```bash
update mysql.user set authentication_string=password('passwd') where user='root';
```

让密码生效

```bash
flush privileges;
```



#### 9. 再次修改配置文件

> 注释掉直接进入MySQL这一段

退出mysql

```bash
exit
```

编辑my.cnf配置文件将：skip-grant-tables这一行注释掉

```bash
[mysqld]
#skip-grant-tables
character_set_server=utf8
init_connect='SET NAMES utf8'
```

重启mysql服务

```bash
systemctl stop  mysqld.service
```

```bash
systemctl start mysqld.service
```



#### 10. 重新更新密码

> `mysql -uroot -p`，密码是之前设置的暂时密码，此时进入你需要更新密码，通常来说你自己的密码系统会认为安全性太低不通过，我们先把安全策略降到最低，之后再修改密码

修改validate_password_policy参数的值

```bash
set global validate_password_policy=0;
```

修改validate_password_length参数的值(密码长度)

```bash
set global validate_password_length=1;
```

修改MySQL为自己的密码

```bash
alter user 'root'@'localhost' identified by 'password';
```

查看并设置密码策略

```bash
SHOW VARIABLES LIKE 'validate_password%';
```

```bash
set global validate_password_policy=LOW;
```



#### 11. 开启远程登录

> 先开放服务器的3306端口，再进入MySQL开启远程登录

如果防火墙开了的话就开放服务器端口

```bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

```bash
firewall-cmd --reload
```

进入MySQL，开启远程登录

```bash
grant all privileges on *.* to 'root'@'%' identified by 'passwd' with grant option;
```



> 至此，安装就结束啦！！！
