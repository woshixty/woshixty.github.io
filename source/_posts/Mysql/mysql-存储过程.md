---
title: MySQL存储过程学习
date: 2021-05-25
tags: [mysql]
categories: mysql
---

## 一、变量

### 1、系统变量

系统变量：由系统提供，并不是用户定义的，属于服务层面，由于作用域的不同分为以下两种：

- 全局变量
- 会话变量

#### a、查看所有的系统变量

```sql
SHOW GLOBAL | [SESSION] VARIABLES
-- SESSION（可省略） 或者 GLOBAL
```

执行结果如下：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/mysql1.png"/>



#### b、查看满足条件的系统变量

```sql
SHOW GLOBAL | [SESSION] VARIABLES LIKE '%char%'
```



#### c、查看指定系统变量的值

```sql
SELECT @@GLOBAL | [SESSION].系统变量名
```



#### d、为某个系统变量赋值

- 方式一：

  ```sql
  SET GLOBAL | [SESSION] 系统变量名 = 值
  ```

- 方式二：

  ```sql
  SET @@GLOBAL | [SESSION].系统变量名 = 值
  ```

注意事项：

- 如果是全局级别，则需要加GLOBAL，如果是会话级别，则需要加SESSION，不写默认SESSION
- 全局变量对所有会话均有效，会话变量仅仅对当前会话有效



### 2、自定义变量

由用户自定义，非系统提供的变量

#### a、用户变量

使用步骤：` 声明`、` 赋值`、` 使用（查看、比较、运算等）`；作用域：仅仅对当前会话（连接）有效，同会话变量

##### 1、声明并初始化

赋值的操作符：` =`或者` :=`，注：SELECT只能用` :=`

- 方式一：

  ```sql
  SET @用户变量名=值
  ```

- 方式二：

  ```sql
  SET @用户变量名:=值
  ```

- 方式三：

  ```sql
  SELECT @用户变量名:=值
  ```

##### 2、赋值

可以由上面的三种方式赋值，也可以将查询结果赋值给变量

```sql
-- SELECT 字段 INTO @变量名 FROM 表名
SELECT COUNT(*) INTO @amount FROM activity_check
```

##### 3、使用（查看用户变量）

```sql
SELECT @用户变量名
```



#### b、局部变量

作用域：仅仅定义在它的` begin end块中有效`

##### 1、声明

```sql
DECLARE 变量名 类型;
DECLARE 变量名 类型 DEFAULT 值;
```

##### 2、赋值

与用户变量一致，只不过使用` SET`或者` SELECT INTO`时，无需用@符号

##### 3、使用

```sql
SELECT 局部变量名
```

用户变量与局部变量区别：

| 变量类型 | 作用域        | 定义与使用位置                  | 语法                        |
| -------- | ------------- | ------------------------------- | --------------------------- |
| 用户变量 | 当前会话      | 会话中的任何位置                | 必须加@符号，不用限定类型   |
| 局部变量 | BEGIN END块中 | 只能在BEGIN END中，且为第一句话 | 一般不加@符号，需要限定类型 |



## 二、存储过程



## 三、函数



