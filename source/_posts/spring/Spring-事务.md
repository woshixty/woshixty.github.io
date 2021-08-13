---
title: spring-事务
date: 2021-06-14
tags: [spring]
categories: Spring
---

### 一、什么事务

` 事务`是数据库操作的基本单元，也就说逻辑上的一组操作，要么都成功，如果有一个失败操作则全都失败。



### 二、事务的四个特性（ACID）

- 原子性

  要么都成功，要么都失败

- 一致性

  操作之前和操作之后总量不变

- 隔离性

  多事务操作时不过会相互产生影响

- 持久性

  提交之后数据库中会发生改变



### 三、准备环节

#### 1、创建数据库

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_account
-- ----------------------------
DROP TABLE IF EXISTS `t_account`;
CREATE TABLE `t_account`  (
    `id` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
    `username` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
    `money` int(10) NULL DEFAULT NULL,
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_account
-- ----------------------------
INSERT INTO `t_account` VALUES ('1', 'lusy', 1000);
INSERT INTO `t_account` VALUES ('2', 'mary', 1000);

SET FOREIGN_KEY_CHECKS = 1;
```

#### 2、Service And Dao

> 创建Service，搭建Dao，完成对象创建和注入关系

service注入dao，在dao注入JdbcTemplate，在JdbcTemplate注入DataSource

- service

  ```java
  @Service
  public class UserService {
      //注入Dao
      @Autowired
      private UserDao userDao;
  }
  ```

- dao

  ```java
  public interface UserDao {
  }
  ```

  ```java
  @Repository
  public class UserDaoImpl implements UserDao {
      @Autowired
      private JdbcTemplate jdbcTemplate;
  }
  ```

- bean.xml

  ```xml
  <!-- 组件扫描 -->
      <context:component-scan base-package="com.atguigu"></context:component-scan>
      <!-- 数据库连接池配置 -->
      <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
          <property name="url" value="jdbc:mysql://ip:3306/user_db?useUnicode=true&amp;characterEncoding=UTF-8&amp;useSSL=false&amp;serverTimezone=Asia/Shanghai"></property>
          <property name="username" value="root"></property>
          <property name="password" value="root"></property>
          <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
      </bean>
      <!-- JdbcTemplate对象 -->
      <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
          <!-- 注入dataSource -->
          <property name="dataSource" ref="dataSource"></property>
      </bean>
  ```

#### 3、创建方法

- 在dao中创建两个方法

  ```java
  //多钱
  @Override
  public void add() {
      String sql = "update t_account set money = money + ? where username = ?";
      jdbcTemplate.update(sql, 100, "mary");
  }
  
  //少钱
  @Override
  public void reduce() {
      String sql = "update t_account set money = money - ? where username = ?";
      jdbcTemplate.update(sql, 100, "lucy");
  }
  ```

- 在service添加如下方法模拟转账

  ```java
  //模拟转账的方法
  public void accountMoney() {
      //lucy少100
      userDao.reduce();
      //mary多100
      userDao.add();
  }
  ```

- 添加测试方法

  ```java
  @Test
  public void testAccount() {
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      userService.accountMoney();
  }
  ```

### 四、事务操作过程

#### 1、开启事务

#### 2、进行业务操作

#### 3、没有异常发生就提交事务

#### 4、出现异常就回退





