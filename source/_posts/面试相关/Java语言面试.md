# 1、JDK1.8的新特性有哪些

## 一、接口的默认方法

## 二、Lambda表达式

## 三、函数式接口

## 四、方法与构造函数引用

## 五、Lambda作用域

## 六、访问局部变量

## 七、访问对象字段于静态变量

## 八、访问接口的默认方法

# 2、Java抽象类和接口有什么区别

语法与语意的区别

什么时候用抽象类，什么时候用接口

抽象类 -> 有一种既有抽象概念（植物、动物）

接口 -> 共同特征（例如具有某种会飞的功能）

#### 相同点：

- 都不能被实例化
- 接口的实现类和抽象类的子类都只有在实现了接口或抽象类中的方法以后才能被实例化

#### 不同点：

- 接口中的方法只有定义（JDK1.8中可以有default方法），抽象类的方法可以定义与实现

- 关键字不同：`implement`和`extends`，类可以实现多个接口，但只能继承一个类

- 接口强调功能的实现，抽象类强调所属关系

- 接口中的成员变量默认都是` public static final`，必须初始化，不可被修改

  抽象类的成员变量默认是default，可以被重新定义，也可以被修改

- 接口中的成员方法都是` public、abstract`的

  抽象方法都是要被` abstract`修饰的不能被` private、synchronized、native`等修饰，不带花括号

# 3、HashMap

## （1）底层原理

https://zhuanlan.zhihu.com/p/79507868

## （2）HashMap在扩容上做了哪些处理

JDK1.8不需要再像JDK1.7一样重新计算hash，只需要看原来的hash值新增的那一位bit是1还是0就好

## （3）HashMap有哪些线程安全的方式

- 方法一：

通过`Collections.synchronizedMap()`返回一个新的Map，这个新的Map就是线程安全的。这个要求大家习惯基于接口编程，因为返回的并不是HashMap，而是一个Map实现。

- 方法二：

重新改写了HashMap，具体可以查看`java.util.concurrentHashMap`，是一个支持并发的HashMap。这个方法比方法一有了很大的改进。

# 4、HashMap和HashTable的区别

- HashTable线程同步但效率低，HashMap相反但效率高

  线程同步：指的是多线程的时候数据的安全性，synchronized关键字

- HashTable不允许<key, value>空值，HashMap允许

- HashTable使用Enumeration，HashMap使用Iterator

- HashTable（默认为11）扩容方式：2*old+1，HashMap（默认为16）扩容方式为：2的指数倍

- HashTable继承自Dictionary类，HashMap继承自AbstractMap类

# 5、Comparetor和Comparable

在Java中当自定义的类需要进行比较的时候，可以通过这两种方式

Comparable是一个比较接口，一旦某个类实现该接口，就代表这个类“支持排序比较”，而Comparator是一个比较器，通过建立一个“该类的比较器”来实现排序

Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”

Comparable实现简单，只需实现接口即可进行比较，但是需要修改源码，但是Comparetor好处是不需要修改源码，只是实现一个比较器，将需要比较的两个对象传禁区就行

# 6、线程池的实现原理

https://juejin.cn/post/6866054685044768782

线程池的核心参数：

- `corePoolSize`：线程池核心线程数量
- `maximumPoolSize`：线程池会创建的最大线程数量
- `keepAliveTime`：线程池中大于核心线程数那部分线程的最大空闲存活时间
- `workQueue`：保存等待执行任务的一个阻塞队列，当线程池所有的线程都在运行的时候，提交的任务会保存在阻塞队列中
- `threadFactory`：创建线程的一个工厂，默认为`DefaultThreadFactory`类
- `handler`：线程拒绝策略
