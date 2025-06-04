---
title: maven生命周期、插件、命令
date: 2021-07-09
tags: [maven]
categories: Maven
---

maven的生命周期：项目构建的各个阶段，包括` 清理、编译、测试、报告、打包、安装、部署`

插件：要完成项目的各个阶段，要使用maven命令，执行命令的功能是通过插件完成的。插件就是` jar`，代表一些类。

命令：执行maven功能是由命令发出的，比如` mvn compile`



单元测试（Junit）：

Junit是一个单元测试的工具在Java中经常使用

单元：在Java中指方法，一个方法就是一个单元

作用：使用Junit来测试方法是否达标了



maven相关命令：

（1）mvn clean

（2）mvn compile：编译代码与拷贝文件