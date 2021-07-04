---
title: 数据库概论复习-关系数据库标准语言SQL
date: 2021-06-20
tags: [mysql]
categories: mysql

---

`结构化查询语言（Structure QUery Language，SQL）`是关系数据库的标准语言

### 一、SQL概述

大多数数据库均采用SQL作为共同的数据存取语言和标准接口

#### 1、SQL的产生与发展

目前没有任何一个数据库能支持SQL标准的所有概念特性，SQL标准的发展如下所示：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/table-3-1.png">

#### 2、SQL的特点

它功能极强而又简单易学，集`数据查询（data query）`、` 数据操纵（data manipulation）`、` 数据定义（data definition）`、` 数据控制（data control）`，其特点主要有以下几点：

- 综合统一

  集很多功能于一体，可以独立完成数据库生命周期中的全部活动

- 高度非过程化

  只需提出做什么，无需了解存取路径，存取过程以及SQL的操作有系统完成

- 面向集合的操作方式

  操作对象、查询结果、一次操作（插入、删除、更新）的对象也可以是集合

- 以同一种语法结构提供多种使用方式

  SQL既是` 独立的语言（直接在键盘输入SQL命令进行操作）`，有时` 嵌入式的语言（嵌入到诸如Java、Python的高级语言中）`

- 语法简洁，易学易用

  完成核心动作仅需九个动词，接近英语口语

SQL的动词如下所示：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/table-3-2.png">

#### 3、SQL的基本概念

支持数据库的三级模式结构：`外模式`、`模式`、`内模式`，其中外模式包括：` 视图（view）`、` 部分基本表（base table）`，模式包括` 若干基本表`，内模式包括` 若干存储文件（stored file）`。如下图所示：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/pic-3-1.png">

`基本表`和` 视图`一样，都是` 关系`，用户可以用SQL对` 基本表`和`视图`进行操作。

`基本表`是本身独立存在表，一个` 关系`就对应一个` 基本表`，一个或多个` 基本表`对应一个存储文件，一个表可以带若干索引，索引也存放在存储文件中。

`存储文件`的逻辑结构组成了关系数据库的` 内模式`。存储文件的物理结构对最终用户是隐蔽的

`视图`是从一个或几个基本表导出的表

### 二、学生课程数据库的定义

- Student表

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/sudent.png">

- Course表

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/course.png">

- SC表

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/sc.png">

### 三、数据定义

SQL的数据定义包括` 模式定义`、` 表定义`、` 视图定义`、` 索引定义`，如下图：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E8%AE%BA%E5%A4%8D%E4%B9%A0/table-3-3.png">

SQL不提供修改模式定义和修改视图定义的操作，修改的话只能删除重建。

**一个关系数据库管理系统的实例（instance）中可以建立多个数据库，一个数据库可以建立多个模式，一个模式通常可以包括多个表、视图、索引等数据库对象。**



#### 1、模式的定义与删除

##### （1）定义模式

在SQL中，创建模式的定义语句如下：

> CREATE SCHEMA <模式名> AUTHORIZATION <用户名>;
>
> 如果没有指定<模式名>，则<模式名>隐含为<用户名>

要创建模式，调用该命令的用户必须具有数据库管理员的权限，或者获得了数据库管理员授予的` CREATE SCHEMA`的权限

**例题**

- 为用户WANG定义一个学生-课程模式S-T

  ```mysql
  CREATE SCHEMA "S-T" AUTHORIZATION WANG;
  ```

- 为用户WANG创建一个模式TEST，并在其中定义一个表TAB1

  ```mysql
  CREATE SCHEMA TEST AUTHORIZATION ZHANG
  CREATE TABLE TAB1(
  	COL1 SMALLINT,
    COL2 INT,
    COL3 CHAR(20),
    COL4 NUMBERIC(10, 3),
    COL5 DECIMAL(5, 2)
  );
  ```



##### （2）删除模式

删除模式的语句定义如下：

> DROP TABLE <模式名> <CASCADE | RESTRICT>
>
> 其中` CASCADE`和` RESTRICT`两者必选其一，选择了` CASCADE（级联）`就表示在删除模式时会级联删除该模式下的所有数据库对象；选择了` RESTRICT（限制）`就表示如果模式中已经定义了下属的数据库对象（表、视图等），则拒绝该语句的执行

**例题**

- 删除ZHANG并级联删除该模式中的所有数据库对象

  ```mysql
  DROP SCHEMA ZHANG CASCADE;
  ```



#### 2、基本表的定义、删除与修改

##### （1）定义基本表







