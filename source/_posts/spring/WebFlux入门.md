---
title: spring-WebFlux
date: 2021-07-06
tags: [spring]
categories: Spring
---

### 一、SpringWebFlux介绍

可以直接去官网看看：https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html

（1）是Spring5添加的新的模块，用于web开发的，功能与SpringMVC类似的，WebFlux使用的当前一种比较流行的响应式编程框架

（2）使用传统的web框架，比如SpringMVC，这些基于Servlet容器，WebFlux是一种异步非阻塞的框架，异步非阻塞框架在Servlet3.1以后才支持，核心是基于Reactor相关API实现的

（3）异步非阻塞：

- 异步与同步

  > 针对调用者来说，调用者发送请求，如果等着对方回应之后才去做其他事情就是` 同步`；如果发送请求之后，不等对方回应就去做其他事情就是` 异步`

- 非阻塞与阻塞

  > 针对被调用者而言，被调用者在收到请求之后，做完任务之后才给出反馈就是` 阻塞`；收到请求之后马上给出反馈然后再去做事情就是就是` 非阻塞`

（4）WebFlux的特点：

- 1、非阻塞式：可以在有限的资源下，提高系统的吞吐量和伸缩性，以Reactor为基础实现函数式编程
- 2、Spring5基于Java8，WebFlux使用Java8函数式编程的方式实现路由请求

（5）与SpringMVC比较

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/WebFlux/spring-mvc-and-webflux-venn.png">

- 1、上述两个框架都可以使用注解方式，都运行在Tomcat等容器中
- 2、SpringMVC采用命令式编程，WebFlux采用异步式响应编程

网关使用`异步式响应编程`更好，因为网关需要处理更多的请求



### 二、响应式编程（Java实现）

#### （1）什么是响应式编程

>**简称RP（Reactive Programming）**
>
>响应式编程是一种` 面向数据流`和` 变化传播`的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。

根据数据或者数据流发生变化，相应的计算模型就会根据值的变化而变化，例如：电子表格程序就是响应式编程的一个例子。单元格可以包含字面值或类似"=B1+C1"的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化。

#### （2）Java8及其之前的版本

提供了` 观察者模式`的两个类：` Observer`和` Observable`。即观察你关心的数据的变化，一旦出现变化，就发出通知。

直接上代码，代码中我们设置了改变并进行了通知：

```java
public class ObserverDemo extends Observable {
    public static void main(String[] args) {
        ObserverDemo observer = new ObserverDemo();
        //添加观察者
        observer.addObserver((o, arg) -> {
            System.out.println("发生变化");
        });
        observer.addObserver((o, arg) -> {
            System.out.println("手动改变观察者通知，准备改变");
        });
        //数据变化
        observer.setChanged();
        //通知
        observer.notifyObservers();
    }
}
```

#### （3）Java9及其之后的版本

使用` Flow`实现，这里不做介绍



### 三、响应式编程（Reactor实现）

#### （1）规范框架

响应式编程操作中，` Reactor`是满足` Reactive`规范框架

#### （2）核心类

` Reactor`有两个核心类，`Mono`和` Flux`，这两个类实现接口` Publisher`，提供丰富操作符。` Flux`对象实现发布者，返回N个元素；` Mono`实现发布者，返回0或者一个元素

#### （3）三种信号

` Flux`和` Mono`都是数据流的发布者，使用` Flux`和` Mono`都可以发出三中信号：`元素值`、` 错误信号`、` 完成信号`（错误信号与完成信号用于告诉订阅者数据流结束了，他俩都是终止信号，错误信号在终止数据流时会同时将错误信息传递给订阅者）

以下图片说明了两者的区别（Flux返回0个或者多个元素，Mono返回一个元素）：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/WebFlux/FluxAndMono.jpeg">

#### （4）代码演示

接下来咱们直接在代码上来实现：

- 引入依赖

```xml
<dependency>
  <groupId>io.projectreactor</groupId>
  <artifactId>reactor-core</artifactId>
  <version>3.4.0</version>
</dependency>
```

- 代码编写

```java
public class TestReactor {
    public static void main(String[] args) {
        //just方法直接声明
        Flux.just(1,2,3,4);
        Mono.just(1);
        //其他方法进行声明
        Integer[] array = {1, 2, 3, 4};
        Flux.fromArray(array);

        List<Integer> list = Arrays.asList(array);
        Flux.fromIterable(list);

        Stream<Integer> stream = list.stream();
        Flux.fromStream(stream);
    }
}
```

#### （5）三种信号的特点

- 错误信号和完成信号都是终止信号，是不能共存的
- 如果没有发送任何元素值，而是直接发送错误信号或者完成信号，表示的是空数据流
- 如果没有错误信号，没有完成信号，表示的是无线信号流

#### （6）订阅

调用` just`或者其他方法只是声明数据流，数据流并没有发出，只有进行订阅之后才会触发数据流，不订阅啥都不会发生

```java
Flux.just(1,2,3,4).subscribe(System.out::println);
Mono.just(1).subscribe(System.out::println);
```

#### （7）操作符

对数据流进行一道道操作，成为操作符，比如工厂流水线

- map

  将元素映射为新元素

  <img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/WebFlux/map.png">

- flatMap

  将元素映射为流（将多个元素转成流，然后将多个流转成一个大流）

  <img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/WebFlux/flatMap.png">



### 四、WebFlux执行流程和核心API





### 五、SpringWebFLux（基于注解编程模型）





### 六、SpringWebFLux（基于函数式编程模型）

