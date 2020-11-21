---
title: mysql的安装
date: 2020-11-11
tags: [centos, mysql]
categories: tools
---

(之前安装的MySQL又出问题了，干脆直接重装，解决一切问题)

### MySQL卸载

#### yum

> 使用 yum list installed | grep mysql 全都yum remove掉就可以了

```bash
[root@VM_0_4_centos ~]# yum list installed | grep mysql

mysql-community-client.x86_64        5.7.32-1.el7                    @mysql57-community
mysql-community-common.x86_64        5.7.32-1.el7                    @mysql57-community
mysql-community-libs.x86_64          5.7.32-1.el7                    @mysql57-community
mysql-community-server.x86_64        5.7.32-1.el7                    @mysql57-community
mysql57-community-release.noarch     el7-11                          installed  

[root@VM_0_4_centos ~]# yum remove mysql-community-client.x86_64

Loaded plugins: fastestmirror, langpacks
Resolving Dependencies
--> Running transaction check
---> Package mysql-community-client.x86_64 0:5.7.32-1.el7 will be erased
--> Processing Dependency: mysql-community-client(x86-64) >= 5.7.9 for package: mysql-community-server-5.7.32-1.el7.x86_64
--> Running transaction check
---> Package mysql-community-server.x86_64 0:5.7.32-1.el7 will be erased
--> Finished Dependency Resolution
epel/7/x86_64                                                                                              | 4.7 kB  00:00:00     
epel/7/x86_64/updateinfo                                                                                   | 1.0 MB  00:00:00     
......
Removed:
  mysql-community-client.x86_64 0:5.7.32-1.el7                                                                                    

Dependency Removed:
  mysql-community-server.x86_64 0:5.7.32-1.el7                                                                                    

Complete!
```



#### 赶尽杀绝

> 然后使用 find / -name mysql 依次将文件删除即可

```bash
[root@VM_0_4_centos ~]# find / -name mysql

/var/lib/mysql
/var/lib/mysql/mysql
/usr/share/mysql
```



### 安装MySql

#### 下载安装包

> 先去官网下载安装包，这里我下载的是mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar 然后上传到服务器上

```bash
[root@VM_0_4_centos ~]# ls

log  mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
```



#### 删除系统自带的东西

> 如果是第一次安装，检查一下系统自带的mariadb-lib以及MySQL 我这边干干净净呢

```bash
[root@VM_0_4_centos ~]# rpm -qa|grep mariadb

[root@VM_0_4_centos ~]# rpm -qa|grep mysql
```



#### 解压缩

> 先创建一个目录，然后在目录里 tar -xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar，并赋予目录最大权限

```bash
[root@VM_0_4_centos ~]# mkdir mysql

[root@VM_0_4_centos ~]# cd mysql/

[root@VM_0_4_centos mysql]# tar -xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
mysql-community-embedded-devel-5.7.29-1.el7.x86_64.rpm
mysql-community-test-5.7.29-1.el7.x86_64.rpm
mysql-community-embedded-5.7.29-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.29-1.el7.x86_64.rpm
mysql-community-libs-5.7.29-1.el7.x86_64.rpm
mysql-community-client-5.7.29-1.el7.x86_64.rpm
mysql-community-server-5.7.29-1.el7.x86_64.rpm
mysql-community-devel-5.7.29-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.29-1.el7.x86_64.rpm
mysql-community-common-5.7.29-1.el7.x86_64.rpm

[root@VM_0_4_centos mysql]# cd ..

[root@VM_0_4_centos ~]# chmod -R 777 mysql
```



#### 进行安装

> 严格按照下面四个步骤安装

```bash
[root@VM_0_4_centos mysql]# rpm -ivh mysql-community-common-5.7.29-1.el7.x86_64.rpm 

[root@VM_0_4_centos mysql]# rpm -ivh mysql-community-libs-5.7.29-1.el7.x86_64.rpm 

[root@VM_0_4_centos mysql]# rpm -ivh mysql-community-client-5.7.29-1.el7.x86_64.rpm 

[root@VM_0_4_centos mysql]# rpm -ivh mysql-community-server-5.7.29-1.el7.x86_64.rpm 
```



#### 配置数据库

> 修改配置文件

```bash
[root@VM_0_4_centos mysql]# vim /etc/my.cnf

#在[mysqld]下面添加这三行
#skip-grant-tables  跳过登录验证
#character_set_server=utf8  设置默认字符集UTF-8
#init_connect='SET NAMES utf8'  设置默认字符集UTF-8
```



#### 启动MySQL服务

> 执行下面两个命令

```bash
[root@VM_0_4_centos mysql]# systemctl start mysqld.service
[root@VM_0_4_centos mysql]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```



#### 设置密码

> 设置密码并且让他生效

```bash
mysql> update mysql.user set authentication_string=password('passwd') where user='root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

> 退出mysql并重启mysql服务

```bash
mysql> exit
Bye

[root@VM_0_4_centos mysql]# systemctl stop  mysqld.service

[root@VM_0_4_centos mysql]# systemctl start mysqld.service
```

> 编辑my.cnf配置文件将：skip-grant-tables这一行注释掉

```bash
[mysqld]
#skip-grant-tables
character_set_server=utf8
init_connect='SET NAMES utf8'
```

> 此时你进入需要重新设置一个密码，先设置一个复杂的密码，然后设置密码策略，改成自己的密码

```bash
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)

mysql> set global validate_password_policy=LOW;
Query OK, 0 rows affected (0.00 sec)
```



#### 开启远程登录

> 开放3306端口

```bash
[root@VM_0_4_centos mysql]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
[root@VM_0_4_centos mysql]# firewall-cmd --reload
success
```

> 开启远程登录

```bash
mysql> grant all privileges on *.* to 'root'@'%' identified by 'passwd' with grant option;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```



> 至此，安装就结束啦！！！













