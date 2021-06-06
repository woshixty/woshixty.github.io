---
title: MySQL视图学习
date: 2021-05-24
tags: [mysql]
categories: mysql
---

> MySQL5添加了对视图的支持，那么它到底是个森么东西呢？我们一起来看看吧！

## 一、视图的介绍

### 1、什么是视图？

所谓视图（View），其实是一种虚拟存在的表，同真实表一样，视图也由列和行构成，但视图并不实际存在于数据库中。行和列的数据来自于定义视图的查询中所使用的表，并且还是在使用视图时动态生成的。意思就是：他存的是你写的SQL查询语句。

### 2、使用视图有什么好处？

- 重用SQL语句
- 简化复杂的SQL查询
- 使用表的组成部分而不是整个表
- 保护数据，可以给用户授予表的特定访问权限而不是整个表的访问权限。
- 更改数据的格式和表示

而且在视图创建之后，可以用与表基本相同的方式利用它们（查询、过滤、排序···），并且可以将视图联结到其他视图或表，甚至能添加或更新数据。

> 性能问题：因为视图不包含任何数据，所以每次使用视图，都必须处理查询执行时所需的任一个检索。如果使用多个联结和过滤创建了复杂的视图或者嵌套了视图，可能会发现性能下降的很厉害。

## 二、视图的使用

- 创建：` CREATE VIEW AS`
- 查看：` SHOW CREATE VIEW viewname`
- 删除：` DROP VIEW viewname`
- 更新：先删除再创建，或者` CREATE OR REPLACE VIEW`

## 三、视图的使用案例

先建表：

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for category
-- ----------------------------
DROP TABLE IF EXISTS `category`;
CREATE TABLE `category` (
  `id` int(5) NOT NULL AUTO_INCREMENT COMMENT '商品类别主键',
  `category_name` varchar(255) NOT NULL COMMENT '商品类别名称',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10001 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of category
-- ----------------------------
BEGIN;
INSERT INTO `category` VALUES (1, '电脑组件');
INSERT INTO `category` VALUES (2, '图书文娱');
INSERT INTO `category` VALUES (3, '手机');
COMMIT;

-- ----------------------------
-- Table structure for product
-- ----------------------------
DROP TABLE IF EXISTS `product`;
CREATE TABLE `product` (
  `id` int(5) NOT NULL AUTO_INCREMENT COMMENT '商品表主键，自增',
  `product_name` varchar(255) NOT NULL COMMENT '商品名称',
  `product_category` int(5) NOT NULL COMMENT '商品类别',
  `product_price` decimal(10,2) NOT NULL COMMENT '商品价格',
  PRIMARY KEY (`id`),
  KEY `product_category` (`product_category`),
  CONSTRAINT `product_category` FOREIGN KEY (`product_category`) REFERENCES `category` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of product
-- ----------------------------
BEGIN;
INSERT INTO `product` VALUES (1, 'AMD 锐龙7 5800x', 1, 3199.00);
INSERT INTO `product` VALUES (2, '华硕 TUF-RTX3090-GAMING', 1, 25999.00);
INSERT INTO `product` VALUES (3, 'Java编程思想', 2, 74.50);
INSERT INTO `product` VALUES (4, 'SpringBoot编程思想', 2, 57.80);
INSERT INTO `product` VALUES (5, 'IPhone 12 Pro', 3, 9299.00);
INSERT INTO `product` VALUES (6, '华为Mate 40 pro', 3, 6299.00);
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

### 1、创建视图

```mysql
CREATE VIEW pro_cate AS
SELECT product_name, category.category_name, product.product_price
FROM product
INNER JOIN category ON product.product_category = category.id
```

### 2、查看视图

```mysql
SHOW CREATE VIEW  pro_cate
```

查询结果如下所示：

> - View：` pro_cate`
>
> - Create View：` CREATE ALGORITHM=UNDEFINED DEFINER=`root`@`%` SQL SECURITY DEFINER VIEW `pro_cate` AS select `product`.`product_name` AS `product_name`,`category`.`category_name` AS `category_name`,`product`.`product_price` AS `product_price` from (`product` join `category` on((`product`.`product_category` = `category`.`id`)))`
>
> - collation_connection：` utf8mb4_general_ci`
>
> - character_set_client：` utf8mb4`

### 3、更新视图

```mysql
CREATE OR REPLACE VIEW pro_cate AS
SELECT product.id, product.product_name, category.category_name, product.product_price
FROM product
INNER JOIN category ON product.product_category = category.id
```

### 4、删除视图

```mysql
DROP VIEW pro_cate
```

## 四、在视图里增删改数据

不建议！

