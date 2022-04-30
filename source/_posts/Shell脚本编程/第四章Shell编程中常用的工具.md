---
title: Shell编程中常用的工具
date: 2021-11-26
tags: [shell]
categories: shell
---

### 一、文件查找之find命令

#### 1、语法格式

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Shell%E8%84%9A%E6%9C%AC%E7%BC%96%E7%A8%8B/18find%E5%91%BD%E4%BB%A4%E8%AF%AD%E6%B3%95%E6%A0%BC%E5%BC%8F.png">

 #### 2、选项参数对照表

<center>表格一<center>

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Shell%E8%84%9A%E6%9C%AC%E7%BC%96%E7%A8%8B/19%E9%80%89%E9%A1%B9%E5%8F%82%E6%95%B0%E5%AF%B9%E7%85%A7%E8%A1%A8.png">

<center>表格二<center>
<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Shell%E8%84%9A%E6%9C%AC%E7%BC%96%E7%A8%8B/20%E9%80%89%E9%A1%B9%E5%8F%82%E6%95%B0%E5%AF%B9%E7%85%A7%E8%A1%A82.png">

#### 3、示例实操

- 查找` /etc`目录下以conf结尾的文件

  ```shell
  find /etc -name '*.conf'
  ```

- 查找`/etc`目录下大于1M的文件

  ```shell
  find /etc -size +
  ```

- 从二级子目录开始查找文件（f）/目录（d）

  ```shell
  find . -mindepth 2 -type f

#### 4、操作

#### （1）-print

打印输出，默认选项

#### （2）-exec

对搜索到的文件执行特定的操作，格式为` -exec 'command {} \;'`

- 搜索`/etc`下的文件（非目录），文件以conf结尾，且大于10k，然后将其删除

  ```shell
  find ./etc/ -type f -name '*.conf' -size +10k -exec rm -f {} \;
  ```

- 搜索条件和例一一样，将其复制到`/root/conf`目录下

  ```shell
  find ./etc/ -size +10k -type f name '*.conf' -exec cp {} /root/conf/ \;
  ```

- 将`/var/log`目录下以'*.log'结尾的文件且更改时间在七天以上的删除

  ```shell
  find /var/log/ -name '*.log' -mtime +7 -exec rn -rf {} \;
  ```

##### （3）-ok

和exec功能一样，但是每次操作都会给用户提示

#### 5、逻辑运算符

- -a

  与

- -o

  或

- -not / !

示例：查找当前目录下，属主不是hdfs的所有文件

```shell
find . -not -user hdfs | find . ! -user hdfs
```



#### 6、find、locate、whereis和which总结及适用场景分析

##### （1）locate

- 文件查找命令，所属软件包是`mlocate`
- 不同于`find`命令是在整块磁盘中搜索，`locate`命令是在数据库文件中查找
- `find`默认是全部匹配，`locate`是默认匹配部分

通过`updatedb`命令更新数据库

##### （2）whereis

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Shell%E8%84%9A%E6%9C%AC%E7%BC%96%E7%A8%8B/whereis.png">

##### （3）which

##### 作用：只返回二进制文件

##### （4）比较分析

| 命令    | 场景                               | 分析                     |
| ------- | ---------------------------------- | ------------------------ |
| find    | 查找某一类文件，比如文件名部分一致 | 功能强大，速度慢         |
| locate  | 只查找单个文件                     | 功能强大，速度快         |
| whereis | 查找程序的可执行文件、帮助文档等   | 不常用                   |
| which   | 只查找程序的可执行文件             | 常用于查找程序的绝对路径 |

