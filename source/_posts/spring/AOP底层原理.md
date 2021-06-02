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



