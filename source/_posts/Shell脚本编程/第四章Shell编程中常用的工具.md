---
 title: Shell编程中常用的工具
date: 2021-11-26
tags: [shell]
categories: shell
---

### 一、文件查找之find命令（上）

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

- 查找大于1M的文件

  ```shell
  find /etc -size +
  ```

  