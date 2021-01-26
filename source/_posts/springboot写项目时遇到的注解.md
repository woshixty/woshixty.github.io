---
title: springboot写项目时遇到的各种注解
date: 2020-11-15
tags: [Java, springboot]
categories: springboot
---

> 平常经常使用springboot注解·，但是并不理解
>
> 故做一点总结

### @Data 注解

> @Data 注解的主要作用是提高代码的简洁，使用这个注解可以省去代码中大量的get()、 set()、 toString()等方法；

 @Data 注解要先引入lombok，lombok 是什么，它是一个工具类库，可以用简单的注解形式来简化代码，提高开发效率。

 原理：
- Lombok本质上就是一个实现了“JSR 269 API”的程序。在使用javac的过程中，它产生作用的具体流程如下
- javac对源代码进行分析，生成了一棵抽象语法树（AST）
- 运行过程中调用实现了“JSR 269 API”的Lombok程序
- 此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和		  setter方法定义的相应树节点
- javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）

优点：
能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，提高了一定的开发效率
让代码变得简洁，不用过多的去关注相应的方法
属性做修改时，也简化了维护为这些属性所生成的getter/setter方法等

缺点：
不支持多种参数构造器的重载
虽然省去了手动创建getter/setter方法的麻烦，但大大降低了源代码的可读性和完整性，降低了阅读源代码的舒适度



>  常用的几个注解：
> @Data ： 注在类上，提供类的get、set、equals、hashCode、canEqual、toString方法
> @AllArgsConstructor ： 注在类上，提供类的全参构造
> @NoArgsConstructor ： 注在类上，提供类的无参构造
> @Setter ： 注在属性上，提供 set 方法
> @Getter ： 注在属性上，提供 get 方法
> @EqualsAndHashCode ： 注在类上，提供对应的 equals 和 hashCode 方法
> @Log4j/@Slf4j ： 注在类上，提供对应的 Logger 对象，变量名为 log



### @Repository @Controller @Service  @Component 四大注解

> 依然不是很理解这四大注解的作用, 于是下面摘抄了一些笔记

1、@controller 控制器（注入服务）
用于标注控制层，相当于struts中的action层

2、@service 服务（注入dao）
用于标注服务层，主要用来进行业务的逻辑处理

3、@repository（实现dao访问）
用于标注数据访问层，也可以说用于标注数据访问组件，即DAO组件.

4、@component （把普通pojo实例化到spring容器中，相当于配置文件中的 <bean id="" class=""/>
泛指各种组件，就是说当我们的类不属于各种归类的时候（不属于@Controller、@Services等的时候），我们就可以使用@Component来标注这个类。



### @Autowired 注解

> @Autowired 是一个注释，它可以对类成员变量、方法及构造函数进行标注，让 spring 完成 bean 自动装配的工作。
>
> @Autowired 默认是按照类去匹配，配合 @Qualifier 指定按照名称去装配 bean。

通俗理解就是 这个注解就是spring可以自动帮你把bean里面引用的对象的setter/getter方法省略，它会自动帮你set/get。

常见用法

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import blog.service.ArticleService;
import blog.service.TagService;
import blog.service.TypeService;
 
@Controller
public class TestController {
 
	//成员属性字段使用 @Autowired，无需字段的 set 方法
	@Autowired
	private TypeService typeService;
	
	
	//set 方法使用 @Autowired
	private ArticleService articleService;
	@Autowired
	public void setArticleService(ArticleService articleService) {
		this.articleService = articleService;
	}
 
	//构造方法使用 @Autowired
	private TagService tagService;
	@Autowired
	public TestController(TagService tagService) {
		this.tagService = tagService; 
	}
}
```



### @RestController @Controller @ResponseBody 注解

> 在Spring中@RestController的作用等同于@Controller + @ResponseBody。
>
> 所以想要理解@RestController注解就要先了解@Controller和@ResponseBody注解。

在一个类上添加@Controller注解，表明了这个类是一个控制器类。但想要让这个类成为一个处理请求的处理器光有@Controller注解是不够的，他还需要进一步修炼才能成为一个处理器。

- 第一种方式在spring容器中创建该类的实例
  `<bean class="test.controller.MyController" /> `

  上述这种方式是在spring容器中注入单个bean，当项目比较大，控制器类比较多时，用这种方式向Spring容器中注入bean非常的让人苦恼，索性有第二种方式。

- 第二种方式在spring容器中创建该类的实例
  `<context:component-scan base-scan="test.controller" /> `

  这种方式会扫描指定包中的所有类，并生成相应的bean注入到spring容器中。使用这种方式当然能够极大提高我们的开发效率，但是有时候我们不想某一类型的类注入到spring容器中，这个时候第下面种方式可以解决。

  ```java
  <context:component-scan base-package="test" >
    　　<context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
  </context:component-scan>
  ```

  上述代码表示扫描test包中除有@Service注解之外的类

> 将@Controller注解的类注入Spring容器中，只是该类成为处理器的第一步，想要修炼大成，还需要在该类中添加注解@RequestMapping。

- @RequestMapping

@RequestMapping注解是用来映射请求的，即指明处理器可以处理哪些URL请求，该注解既可以用在类上，也可以用在方法上。

当使用@RequestMapping标记控制器类时，方法的请求地址是相对类的请求地址而言的；当没有使用@RequestMapping标记类时，方法的请求地址是绝对路

径；@RequestMapping的地址可以是uri变量，并且通过@PathVariable注解获取作为方法的参数。也可以是通配符来筛选请求地址。

- @ResponseBody注解

@ResponseBody表示方法的返回值直接以指定的格式写入Http response body中，而不是解析为跳转路径。

格式的转换是通过HttpMessageConverter中的方法实现的，因为它是一个接口，因此由其实现类完成转换。

如果要求方法返回的是json格式数据，而不是跳转页面，可以直接在类上标注@RestController，而不用在每个方法中标注

@ResponseBody，简化了开发过程。





### @Component

组件，放在类上，说明这个类呗spring管理了，即bean。他有几个衍生注解，在web开发中会按照三层架构分层，功能类似，代表被spring托管

- dao【@Repository】

- service【@Service】

- controller 【@Controller】

  四个注解功能一样





### @Autowried

@Autowried通过类型名字自动装配，如果@Autowried不能自动装配属性，就用@Qualifier(value = "xxx")匹配





### @Resource

自动通过名字类型装配





### @Scope("singleton")

单例、原型等等模式





### @Aspect

标注这个类是一个切面类





### @Before("execution(* ...)")

before方法





