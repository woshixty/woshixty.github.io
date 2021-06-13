---
title: spring-aop底层原理
date: 2021-06-02
tags: [spring, aop]
categories: Spring
---

https://cos-1301609895.cos.ap-nanjing.myqcloud.com/aop4.png

### 一、AOP底层使用动态代理

#### 1、两种动态代理

##### a、有接口

需使用JDK的动态代理，创建接口实现类的代理对象，增强类的方法，如下图：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/aop1.png">

##### b、没有接口

使用CGLIB动态代理，继承父类重写父类方法，实现代理，如下图：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/aop2.png">



### 二、AOP（JDK动态代理）

#### 1、使用JDK动态代理，使用Proxy类里的方法创建代理对象

> Class Proxy
>
> - [java.lang.Object](https://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/Object.html)
>   - java.lang.reflect.Proxy

使用下面这个方法创建代理类的实例：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/aop4.png">

` newProxyInstance`方法三个参数的含义：

- ` ClassLoader loader`：类加载器
- ` 类<?>[] interfaces`：增强方法所在的类，这个类所实现的接口
- ` InvocationHandler h`：实现这个接口InvocationHandler，创建代理对象，写增强的方法

#### 2、使用JDK动态代理来写代码

##### a、创建接口，定义方法

```java
public interface UserDao {
    public int add(int a, int b);
    public String update(String id);
}
```

##### b、创建接口实现类，实现方法

```java
public class UserDaoImpl implements UserDao {
    @Override
    public int add(int a, int b) {
        return a + b;
    }
    @Override
    public String update(String id) {
        return id;
    }
}
```

##### c、使用Proxy类创建接口代理对象

匿名内部类方式：

```java
public class JDKProxy {
    public static void main(String[] args) {
        //创建接口实现类代理对象
        Class[] interfaces = {UserDao.class};
        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                return null;
            }
        });
    }
}
```

实现接口的方式：

```java
public class JDKProxy {
    public static void main(String[] args) {
        //创建接口实现类代理对象
        Class[] interfaces = {UserDao.class};
        UserDao userDao = new UserDaoImpl();
        UserDao dao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
        int result = dao.add(1, 2);
        System.out.println(result);
    }
}

//创建代理对象代码
class UserDaoProxy implements InvocationHandler {
    private Object object;
    //创建的是谁的代理对象，就将谁传进来
    //有参构造传递参数
    public UserDaoProxy(Object obj) {
        this.object = obj;
    }

    //写我们要增强的操作逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //方法之前
        System.out.println("方法之前执行......" + method.getName() + "：传递的参数是" + Arrays.toString(args));
        //被增强的方法
        Object res = method.invoke(object, args);
        //方法之后
        System.out.println("方法之前执行......" + object);
        return res;
    }
}
```



### 三、AOP（术语）

#### 1、连接点

类里面哪些方法可以被增强，这些方法称为连接点

#### 2、切入点

实际真正被增强的方法，这些方法称为切入点

#### 3、通知（增强）

实际增强的逻辑部分，即增加的代码逻辑部分，通知有以下几种：

- 前置通知
- 后置通知
- 环绕通知
- 异常通知
- 最终通知

#### 4、切面

指的是一个动作，指把通知应用到切入点的过程



### 四、AOP操作（准备）

Spring框架一般都是基于AspectJ实现AOP操作的

#### 1、什么是AspectJ

AspectJ不是Spring的组成部分，它是一个独立的AOP框架，一般将Spring和AspectJ一起使用，从而进行AOP操作

#### 2、基于AspectJ实现AOP操作

##### a、基于xml配置文件方式实现

##### b、基于注解方式实现（使用）

#### 3、在项目工程中引入相关AOP依赖

需要引入以下相关依赖：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/aop5.png">



#### 4、切入点表达式

##### a、切入点表达式的作用

知道对哪个类里面的那个方法进行增强

##### b、语法结构

execution( [权限修饰符] [返回类型] [类全路径]  [方法名称]  ( [参数列表] ) )

- 对一个类里的某一个方法进行增强

  ```
  
  ```

- 对一个类里的所有的方法进行增强

  ```java
  
  ```

- 对包里的所有类，类里面的所有方法进行增强

  ```java
  ```



### 五、AOP操作（AspectJ注解）

#### 1、创建类，在类中定义方法

```java
public class User {
    public void add() {
        System.out.println("add()......");
    }
}
```

#### 2、创建增强类（编写增强逻辑）

在增强类里面创建方法，让不同的方法代表不同的通知

```java
//增强的类
public class UserProxy {
    //前置通知
    public void before() {
        System.out.println("before()......");
    }
}
```

#### 3、进行通知的配置

##### a、在Spring配置文件中，开启注解扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>
</beans>
```

##### b、使用注解创建User和UserProxy对象

给` User`和` UserProxy`加上` @Component`

##### c、在增强的类上面添加注解@Aspect

` @Aspect`代表生成的对象

##### d、在Spring配置文件中开启生成代理对象

```xml
<!-- 开启Aspect生成代理对象 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```



#### 4、配置不同类型的通知

在增强类里面，在作为通知方法最上面添加通知类型注解，使用切入点表达式配置

如下在` UserProxy`类中加上` @Before`注解：

```java
//@Before表示作为前置通知（切入点表达式指明哪个类中的哪个方法）
@Before(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
```

接下来我们看看所有的通知类型：

```java
//增强的类
@Component
@Aspect
public class UserProxy {
    //前置通知
    //@Before表示作为前置通知（切入点表达式指明哪个类中的哪个方法）
    @Before(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
    public void before() {
        System.out.println("before()......");
    }
		//后置通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
    public void after() {
        System.out.println("after()......");
    }
		//最终通知
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
    public void afterReturning() {
        System.out.println("afterReturning()......");
    }
		//异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
    public void afterThrowing() {
        System.out.println("afterThrowing()......");
    }
		//环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前......");
        //被增强的方法
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后......");
    }
}
```

注意点：

- ` @After`和` @AfterReturning`的区别

  前者在方法之后执行，后者在方法返回结果之后执行

- ` @AfterThrowing`是异常通知，只有在发生异常之后才会执行

#### 5、相同切入点的抽取

如上述代码的` "execution(* com.atguigu.spring5.aopanno.User.add())"`部分，具体如何做请看代码：

```java
    //相同切入点抽取
    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add())")
    public void pointdemo() { }

    //前置通知
    //@Before表示作为前置通知（切入点表达式指明哪个类中的哪个方法）
    @Before(value = "pointdemo()")
    public void before() {
        System.out.println("before()......");
    }
```

#### 6、增强类的优先级

当有多个增强类对同一个方法进行增强时，我们可以设置增强类的优先级。在增强类上面添加注解` @Order(数字类型值)`，数字类型值越小代表优先级越高，如下` PersonProxy`的优先级为1：

```java
@Component
@Aspect
@Order(1)
public class PersonProxy {
  ······
}
```

#### 7、完全注解开发

创建配置类，不再需要配置文件，只需创建如下配置类：

```java
@Configuration
@ComponentScan(basePackages = {"com.atguigu"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
}
```





### 六、AOP操作（AspectJ配置文件）

未完待续......





