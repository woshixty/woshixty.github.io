---
title: spring-JdbcTemplate
date: 2021-06-09
tags: [spring]
categories: Spring
---

### 一、什么是JdbcTemplate

所谓` JdbcTemplate`，就是Spring框架对JDBC进行封装，使用` JdbcTemplate`方便实现对数据库的操作

### 二、准备工作

#### 1、引入相关Jar包

链接: https://pan.baidu.com/s/1hWsFROhJlzulOZpnuUgRHw  密码: 2q9k



#### 2、数据库连接池配置

在` src`目录下新建` bean1.xml`文件，在里面加上以下配置信息：

```xml
<!-- 数据库连接池配置 -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
  <property name="url" value="jdbc:mysql:///user_db"></property>
  <property name="username" value="root"></property>
  <property name="password" value="root"></property>
  <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
</bean>
```



#### 3、JdbcTemplate

配置JdbcTemplate对象，注入DataSource

```xml
<!-- JdbcTemplate对象 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  <!-- 注入dataSource -->
  <property name="dataSource" ref="dataSource"></property>
</bean>
```



#### 4、创建Service And Dao

创建service类，创建dao类，在dao中注入jdbcTemplate对象。

##### a、配置组件扫描

```xml
<!-- 组件扫描 -->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

##### b、创建BookService

```java
@Service
public class BookService {
    //注入BookDao
    @Autowired
    private BookDao bookDao;
}
```

##### c、注入JdbcTemplate

```java
@Repository
public class BookDaoImpl implements BookDao {
    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

##### d、创建数据库表

```mysql
```



#### 5、创建Entity实体类

```java
public class Book {
    private Integer userId;
    private String username;
    private String ustatus;
    //getter and setter 方法
}
```



#### 6、Dao

在Dao进行数据库添加操作，调用JdbcTemplate对象里的update方法实现添加操作：

` int update(String var1, @Nullable Object... var2) throws DataAccessException;`：

参数说明：

- 第一个参数：需要执行的sql语句
- 第二个参数：可变参数，设置sql语句值

代码如下：

```java
//添加的方法
@Override
public void add(Book book) {
  //创建SQL语句
  String sql = "insert into t_book values(?, ?, ?)";
  //调用方法实现
  Object[] args = {book.getUserId(), book.getUsername(), book.getUstatus()};
  int update = jdbcTemplate.update(sql, args);
  System.out.println(update);
}
```



#### 7、测试一下

```java
public class TestBook {
    @Test
    public void testAdd() {
        ApplicationContext context =
                new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        Book book = new Book();
        book.setUserId(1);
        book.setUsername("Java");
        book.setUstatus("sale");
        bookService.addBook(book);
    }
}
```

删除与修改与之类似





