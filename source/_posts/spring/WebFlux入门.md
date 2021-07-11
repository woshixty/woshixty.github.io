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

使用` Flow`实现，这里直接看下面代码：

```java
//创建我自己的订阅者
class MySubscriber<T> implements Flow.Subscriber {
    //发布者与订阅者之间的纽带
    private Flow.Subscription subscription;
    //订阅时会触发该问题
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        //开始请求数据
        subscription.request(1);
    }
    //接收到数据是会触发该方法
    @Override
    public void onNext(Object item) {
        System.out.println("获取到数据" + item);
        //请求下一条数据
        subscription.request(1);
    }
    //订阅出错时
    @Override
    public void onError(Throwable throwable) {
        throwable.printStackTrace();
        synchronized ("java") {
            "java".notifyAll();
        }
    }
    //订阅结束时触发该方法
    @Override
    public void onComplete() {
        System.out.println("订阅结束时");
        synchronized ("java") {
            "java".notifyAll();
        }
    }
}
public class PubSubTest {
    public static void main(String[] args) {
        //创建一个SubmissionPublisher作为发布者
        SubmissionPublisher<String> publisher = new SubmissionPublisher<>();
        //创建订阅者
        MySubscriber<String> subscriber = new MySubscriber<>();
        //注册订阅者
        publisher.subscribe(subscriber);
        //发布几个数据项
        System.out.println("开发发布数据...");
        List.of("Java", "Go", "Erlang", "Swift", "Lua")
                .forEach(im -> {
                    //提交数据
                    publisher.submit(im);
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
        //发布结束
        publisher.close();
        //发布结束之后，为了让线程不会死亡，暂停线程
        synchronized ("java") {
            try {
                "java".wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```





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

SpringWebFlux基于Reactor，默认使用的容器是Netty，Netty是高性能的NIO框架，异步非阻塞的框架

#### （1）Netty

- NIO（非阻塞）
- BIO（阻塞）

#### （2） SpringWebFlux执行过程和SpringMVC相似的

- ` SpringWebFlux`核心控制器` DispatchHandler`，实现接口` WebHandler`

- 接口` WebHandler`有一个方法：

  > ```java
  > public interface WebHandler {
  >     Mono<Void> handle(ServerWebExchange var1);
  > }
  > ```

#### （3）SpringWebFlux里面的DispatcherHandler，负责请求的处理

- HandlerMapping：请求查询到处理的方法
- HandlerAdapter：真正负责请求处理
- HandlerResultHandler：响应结果处理

#### （4）SpringWebFlux函数式编程

` SpringWebFlux`实现函数式编程，有两个接口：` RouterFunction（路由处理）`和` HandlerFunction（处理函数）`



### 五、SpringWebFLux（基于注解编程模型）

` SpringWebFlux`实现方式有两种：` 注解编程模型`和` 函数式编程模型`

使用注解编程模型方式，和之前的` SpringMVC`使用相似，只需要将相关依赖配置到项目中，` SpringBoot`自动配置相关运行容器，默认情况下使用Netty服务器

#### （1）创建SpringBoot工程，引入` WebFlux`依赖

 依赖如下：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

##### （2）配置启动端口号

```properties
server.port=8081
```

#### （3）创建包和相关类

- 实体类

  ```java
  public class User {
      private String name;
      private String gender;
      private Integer age;
  }
  ```

- 接口定义操作

  ```java
  public interface UserService {
      //根据ID查询用户
      Mono<User> getUserById(int id);
      //查询所有用户
      Flux<User> getAllUser();
      //添加用户
      Mono<Void> saveUserInfo(Mono<User> user);
  }
  ```

- 接口的实现类

  ```java
  @Service
  public class UserServiceImpl implements UserService {
      //创建map集合存储数据
      private final Map<Integer,User> users = new HashMap<>();
  
      //无参构造
      public UserServiceImpl() {
          this.users.put(1,new User("lucy","nan",20));
          this.users.put(2,new User("mary","nv",30));
          this.users.put(3,new User("jack","nv",50));
      }
  
      //根据id查询
      @Override
      public Mono<User> getUserById(int id) {
          return Mono.justOrEmpty(this.users.get(id));
      }
  
      //查询多个用户
      @Override
      public Flux<User> getAllUser() {
          return Flux.fromIterable(this.users.values());
      }
  
      //添加用户
      @Override
      public Mono<Void> saveUserInfo(Mono<User> userMono) {
          return userMono.doOnNext(person -> {
              //向map集合里面放值
              int id = users.size()+1;
              users.put(id,person);
          }).thenEmpty(Mono.empty());
      }
  }
  ```

- 编写Controller层

  ```java
  @RestController
  public class UserController {
  
      //注入service
      @Autowired
      private UserService userService;
  
      //id查询
      @GetMapping("/user/{id}")
      public Mono<User> geetUserId(@PathVariable int id) {
          return userService.getUserById(id);
      }
  
      //查询所有
      @GetMapping("/user")
      public Flux<User> getUsers() {
          return userService.getAllUser();
      }
  
      //添加
      @PostMapping("/saveuser")
      public Mono<Void> saveUser(@RequestBody User user) {
          Mono<User> userMono = Mono.just(user);
          return userService.saveUserInfo(userMono);
      }
  }
  ```

  说明：

  - ` SpringMVC`方式，同步阻塞的方式，基于` SpringMVC + Servlet + Tomcat`
  - ` SpringWebFlux`方式，异步非阻塞的方式，基于` SpringWebFlux + Reactor + Netty`



### 六、SpringWebFLux（基于函数式编程模型）

- 在使用函数式编程模型操作的时候，需要自己初始化服务器

- 基于函数式编程模型的时候，有两个核心接口：

  ` RouterFunction（实现路由的功能）`和` HandlerFunction（处理请求生成响应的函数）`

  核心任务：定义两个函数式接口的实现并且启动需要的服务器

- ` SpringWebFlux`请求和响应不再是` ServletRequest`和` ServletResponse`

#### （1）创建Handler（具体实现方法）

```java
public class UserHandler {
    private final UserService userService;
    public UserHandler(UserService userService) {
        this.userService = userService;
    }
    //根据ID查询
    public Mono<ServerResponse> getUserById(ServerRequest request) {
        //获取id值
        int userId = Integer.parseInt(request.pathVariable("id"));
        //空值处理
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();
        //调用Service方法得到数据
        Mono<User> userMono = this.userService.getUserById(userId);
        //将userMono进行转换返回，使用Reactor的操作符flatMap
        return userMono.flatMap(person -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                .body(fromObject(person))
        ).switchIfEmpty(notFound);
    }

    //查询所有
    public Mono<ServerResponse> getAllUsers() {
        //调用Service得到结果
        Flux<User> users = this.userService.getAllUser();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(users, User.class);
    }

    //添加
    public Mono<ServerResponse> saveUser(ServerRequest request) {
        //得到User对象
        Mono<User> userMono = request.bodyToMono(User.class);
        return ServerResponse.ok().build(this.userService.saveUserInfo(userMono));
    }
}
```

#### （2）初始化服务器，编写Router

```java
public class Server {
    //1、创建路由
    public RouterFunction<ServerResponse> routingFunction() {
        //创建Handler对象 
        UserService userService = new UserServiceImpl();
        UserHandler handler = new UserHandler(userService);
        //设置路由
        return RouterFunctions.route(
                GET("/users/{id}").and(accept(MediaType.APPLICATION_JSON)), handler::getUserById
        ).andRoute(
                GET("/users").and(accept(MediaType.APPLICATION_JSON)), handler::getAllUsers
        );
    }
}
```

#### （3）创建服务器完成适配

```java
public class Server {
    //1、创建路由
    public RouterFunction<ServerResponse> routingFunction() {
        //创建Handler对象
        UserService userService = new UserServiceImpl();
        UserHandler handler = new UserHandler(userService);
        //设置路由
        return RouterFunctions.route(
                GET("/users/{id}").and(accept(MediaType.APPLICATION_JSON)), handler::getUserById
        ).andRoute(
                GET("/users").and(accept(MediaType.APPLICATION_JSON)), handler::getAllUsers
        );
    }

    //2、创建服务器，完成适配
    public void createReactorServer() {
        //路由和handler适配
        RouterFunction<ServerResponse> route = routingFunction();
        HttpHandler httpHandler = toHttpHandler(route);
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);
        //创建服务器
        HttpServer httpServer = HttpServer.create();
        httpServer.handle(adapter).bindNow();
    }
}
```

#### （4）最终调用

```java
public static void main(String[] args) throws IOException {
  Server server = new Server();
  server.createReactorServer();
  System.out.println("enter to exit");
  System.in.read();
}
```

#### （5）使用WebClient调用

```java
public class Client {
    public static void main(String[] args) {
        //调用服务器地址
        WebClient webClient = WebClient.create("http://127.0.0.1:56264");
        //根据id查询
        String id = "1";
        User user = webClient.get().uri("/users/{id}", id).accept(MediaType.APPLICATION_JSON).
                retrieve().bodyToMono(User.class).block();
        System.out.println(user.getName());
        //查询所有
        Flux<User> results = webClient.get().uri("/users").accept(MediaType.APPLICATION_JSON).
                retrieve().bodyToFlux(User.class);
        results.map(stu -> stu.getName()).buffer().doOnNext(System.out::println).blockFirst();
    }
}
```





