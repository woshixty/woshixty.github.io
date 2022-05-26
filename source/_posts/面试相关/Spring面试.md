# 一、BeanFactory和ApplicationContext的区别

https://juejin.cn/post/6844903877574131726

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/BeanFactory%E5%92%8CApplicationContext%E7%9A%84%E5%8C%BA%E5%88%AB1.jpg" align=left width=800>

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/BeanFactory%E5%92%8CApplicationContext%E7%9A%84%E5%8C%BA%E5%88%AB2.jpg" align=left  width=800>

# 二、Spring AOP

[Spring AOP 扫盲 ](https://www.cnblogs.com/cxuanBlog/p/13060510.html)

## 1、什么是Spring AOP

面向切面编程（Aspect-oriented Programming，俗称AOP）提供了一种面向对象编程（Object-oriented Programming，俗称OOP）的补充

面向对象编程最核心的单元是类（class），然而面向切面编程最核心的单元是切面（Aspects）

与面向对象的顺序流程不同，AOP采用的是横向切面的方式，注入与主业务流程无关的功能，例如事务管理和日志管理

AOP 是一种编程范式，通过**预编译方式和运行期动态代理**实现程序功能的统一维护的一种技术

## 2、相关概念

- 切面（Aspect）
- 连接点（Join Point）
- 通知（Advice）
- 切入点（Pointcut）
- 介绍（Introduction）
- 目标对象（Target Object）
- AOP代理（AOP proxy）
- 织入（Weaving）

## 3、通知分类

- 前置通知（Before Advice）

  在目标方法被调用前调用通知功能；相关的类`org.springframework.aop.MethodBeforeAdvice`

- 后置通知（After Advice）

  在目标方法被调用之后调用通知功能；相关的类`org.springframework.aop.AfterReturningAdvice`

- 返回通知（After-returning）

  在目标方法成功执行之后调用通知功能；

- 异常通知（After-throwing）

  在目标方法抛出异常之后调用通知功能；相关的类`org.springframework.aop.ThrowsAdvice`

- 环绕通知（Around）

  把整个目标方法包裹起来，在**被调用前和调用之后分别调用通知功能**相关的类

## 4、织入的三种时期

- `编译期:` 切面在目标类编译时被织入，这种方式需要特殊的编译器。**AspectJ 的织入编译器就是以这种方式织入切面的。**
- `类加载期:` 切面在目标类加载到 JVM 时被织入，这种方式需要特殊的类加载器( ClassLoader )，它可以在目标类引入应用之前增强目标类的字节码。
- `运行期:` 切面在应用运行的某个时期被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态创建一个代理对象，**Spring AOP 采用的就是这种织入方式。**

## 5、两种实现方式

静态织入（AspectJ）和动态织入（动态代理）

### （1）静态织入（AspectJ）

### （2）动态织入（动态代理）

Spring AOP 是通过动态代理技术实现的，而动态代理是基于反射设计的

Spring AOP 采用了两种混合的实现方式：JDK 动态代理和 CGLib 动态代理



