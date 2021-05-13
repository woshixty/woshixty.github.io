---
title: Java注解学习
date: 2021-01-26
tags: [java]
categories: Java
---

> 当初学习Java的时候并没有特别去学习和理解注解（Annotation），写了一段时间springboot项目之后，准备回来重修。

### 一、注解（Annotation）到底是个啥玩意？

所谓注解（Annotation），其实就是代码里的特殊标记，能够在代码编译、类加载、运行的时候被读取并对它进行相应的处理。说白了其实可以理解为“贴在代码上的便签”，通过使用注解使得程序开发在不改变程序原有逻辑的基础上，在源文件里嵌入一些补充信息。

注解其实可以为程序元素（类、方法、成员变量等）设置元数据，但到底啥是设置元数据，往后面看就行了。

需要注意的一点是，注解实际上并不影响程序执行，但是你若想要它发挥一定的做用，你就得通过配套工具对注解中的信息进行处理，就相当于是你在代码上贴了“标签”，你得写一段程序来检查这个“标签”



### 二、Java的基本注解

#### 1. 限定重写父类方法： @Override注解

> 这个注解我们太熟悉了！强制子类重写父类方法，它能帮助程序员放置一些低级错误：重写的方法名写错了等
>
> 只要重写的方法名不一致就会报错

```java
class father {
    public void info() {
        System.out.println("Hello");
    }
}

class son extends father {
   @Override
   public void info() {
       System.out.println("Java");
   }
}
```



#### 2. Java 9增强的@Deprecated

> 用于标注某个类、方法已过时，当被使用的时候会报警告
>
> 这个注解有两个属性：
>
> - since：该元素哪个版本会过时
> - forRemoval：该元素在将来是否会被删除

```java
// 注意：这个注解时JDK9之后的增强版
@Deprecated( since = "9", forRemoval = true)
class father {
    public void info() {
        System.out.println("Hello");
    }
}
```



#### 3. 抑制编译器警告：@SuppressWarnings 以及 "堆污染"警告与Java 9增强的：@SafeVarargs

> 感觉这俩注解没啥好说的，我们一般也不会用到
>
> - @SuppressWarnings就是可以让程序元素不报警，包括这个程序元素的所有子元素
> - @SafeVarargs防止堆污染



#### 4. 函数式接口：@FunctionalInterface

> 保证某个接口一定是函数式接口，即只含有抽象函数的接口，他只能修饰接口

```java
@FunctionalInterface
interface func {
    
    static void a() {
        System.out.println("a");
    }
    
    default void b() {
        System.out.println("b");
    }
    
    //唯一的抽象方法
    void c();
}
```



### 三、JDK元注解

所谓元注解，就是修饰注解的注解（禁止套娃！），我们先来康康几个常用的注解吧

#### 1. @Retention注解

> @Retention指定被修饰的注解可以存活多长时间，他的value值为以下三个
>
> - `RetentionPolicy.SOURCE`--编译器把注解记录在文件中，当运行Java程序时，JVM不可获取注解信息
>
> - `RetentionPolicy.CLASS`--编译器把注解记录在文件中，当运行Java程序时，JVM可以通过反射获取注解信息
> - `RetentionPolicy.RUNTIME`--只保留在源码中，编译器直接丢弃，例如@data注解，他只是在编译时候让编译器给类加上getter setter之后就被丢弃了

```java
// @data注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Data {
   String staticConstructor() default "";
}
```



#### 2. @Target注解

> @Target指定被修饰的注解能修饰哪些程序单元，这里有很多，就不细讲了，咱们举一个栗子吧！
>
> - `ElementType.TYPE`--可以修饰类、接口（包括注解类型）、枚举定义

```java
// 这是ElementType枚举类内部
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```



#### 3. @Documented注解

> 被@Documented修饰的注解可以被javaDoc工具提取成文档



#### 4. @Inherited注解

> @Inherited修饰的注解具有继承性，如@Inherited修饰了@a注解，而@a修饰了father这个基类，则father的子类son也相当于被@a修饰
>
> 可能有点绕哈，来人！上代码！

定义注解：

```java
// 这是被@Inherited修饰的接口
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface inheritedTest {
}
```

定义类：

```java
@inheritedTest  //@inheritedTest具有继承性 
class father {
    public void info() {
        System.out.println("Hello");
    }
}

// son是father的子类，故son也相当于被@inheritedTest修饰了
class son extends father {
   @Override
   public void info() {
       System.out.println("Java");
   }
}
```





### 四、自定义注解

#### 1. 定义注解

> 其实注解就是接口，所以定义注解和定义接口差不多，在关键字`interface`前加一个`@`即可，如下定义@MyTag注解：

```java
public @interface MyTag {
}
```



#### 2. 注解的成员变量

> 成员变量在注解定义中以无参方法的形式来声明，方法名和返回值分别为变量的名字与类型，如下

```java
public @interface MyTag {
    String name();
    int age();
    String college() default "计算机学院";  // 指定默认值
}
```



#### 3. 提取注解信息

废话不多说，直接上代码！！！

```java
public class getInfoTest {

    @inheritedTest  // 该注解具有继承性
    class father {
        public void info() {
            System.out.println("Hello");
        }
    }

    // 继承father类
    class son extends father {
        @Override
        public void info() {
            System.out.println("Java");
        }
    }

    public static void main(String[] args) throws ClassNotFoundException {
        // 获取son类上面的所有注解
        Annotation[] aArray = Class.forName("son").getAnnotations();
        // 遍历所有注解
        for (Annotation a:aArray) {
            System.out.println(a);
            // 如果注解是@inheritedTest类型
            if (a instanceof inheritedTest)
                // 强转为@inheritedTest类型并取出@inheritedTest中的name值
                System.out.println(((inheritedTest) a).name());
        }
    }
}
```



### 五、自定义注解使用示例

之前写项目遇到过一个问题：如何比较自由的拦截token

以前是直接写一个拦截类然后再写一个配置类从而达到拦截部分请求的目的

使用注解解决方法如下：

#### 1. 定义一个注解

> 定义了注解的目标以及存活时间

```java
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface AuthenToken {
    boolean required() default true;
}
```

#### 2. 注解拦截器

> 拦截器就是处理这个"标签"工具

```java
public class AuthenticationInterceptor implements HandlerInterceptor {

    @Autowired
    StudentDao studentDao;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("token");     //从请求头中获取token
        // 这里就可以写处理逻辑啦！！！
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```



### 六、学无止境

关于Java注解可以学的东西还有很多，这篇文只讲了其中很小一部分，希望继续多多钻研。