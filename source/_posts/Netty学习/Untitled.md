---
title: NIO学习
date: 2022-02-28
tags: [NIO]
categories: Netty
---

## NIO基础

non-blocking IO（非阻塞IO）

### 1、三大组件

#### 1.1、Channel & Buffer

- Channel：

基本概念：

常见的Channel：



- Buffer：

基本概念：

常见的Buffer：



#### 1.2、Selector

基本概念：多线程版本的缺点、线程池版本的缺点、Selector版本的服务器设计



### 2、ByteBuffer

#### 2.1、ByteBuffer正确使用姿势

- 向buffer中写入数据，例如调用channel.read(buffer)
- 调用buffer.flip()切换至**读模式**
- 从buffer读取数据，例如调用buffer.get()
- 调用clear()或compact()切换至**写模式**
- 重复以上步骤



#### 2.2、ByteBuffer结构



#### 2.3、ByteBuffer常见方法



#### 2.4、Scattering Reads

分散读取



### 3、文件编程

#### 3.1、FileChannel

> FileChannel只能工作在阻塞模式下

- 获取

- 读取

- 写入

- 关闭

- 位置

- 大小

- 强制写入



#### 3.2、两个Channel传输数据



#### 3.3、Path

JDK7加入了`Path`和`Paths`类

- `Path`用来表示文件路径
- `Paths`是工具类，用来获取`Path`实例

```java
Path source = Paths.get("Netty学习/netty-demo/data.txt");
```



#### 3.4、Files（工具类）

检查文件是否存在

创建一级目录

创建多级目录

拷贝文件

移动文件

删除文件

删除目录

遍历目录文件

