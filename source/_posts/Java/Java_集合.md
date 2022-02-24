---
title: Java集合
date: 2021-05-25
tags: [java]
categories: Java
---

Java集合大致可分为` Set、List、Queue和Map`四种体系，Java集合就像一种容器，可以将多个对象扔进去。



### 一、Java集合概述

为了保存数量不确定的数据，以及保存具有映射关系的数据（也被称为关联数组），Java提供了集合类。所有的集合类位于` java.util`包下，后来为了处理多线程环境下并发安全的问题，Java5还在` java.util.concurrent`包下提供了一些多线程支持的集合类。

集合类和数组不一样，数组元素既可以是基本类型的值，也可以是对象（实际上保存的是对象的引用）；而集合类只能保存对象（实际上保存的是对象的引用变量，但通常看作对象），

Java集合类主要由两个接口派生出来：

- Collection和Map，Collection和Map是Java集合框架的根接口，这两个接口包含了一些子接口或实现类。如下图就是Collection接口、子接口及其实现类的继承树：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/Collection%E9%9B%86%E5%90%88%E4%BD%93%E7%B3%BB%E7%9A%84%E7%BB%A7%E6%89%BF%E6%A0%91.png">

- 下图是Map体系的继承树，所有的Map实现类用于保存具有映射关系的数据

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/Map%E4%BD%93%E7%B3%BB%E7%9A%84%E7%BB%A7%E6%89%BF%E6%A0%91.png">



从下图可以看出，如果访问List集合中的元素，可以直接根据元素的索引来访问；如果访问Map集合中的元素，可以根据每项元素的key值来访问其value；如果访问Set中的元素，则只能根据元素本身来访问

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/%E4%B8%89%E7%A7%8D%E9%9B%86%E5%90%88%E7%A4%BA%E6%84%8F%E5%9B%BE.png">



### 二、Java11增强的` Collection`和` Iterator`接口

Collection接口是List、Set和Queue的父接口，该接口定义的方法可以自行去查<a href="https://www.matools.com/api/java8">Java API文档</a>，无非就是添加对象、清空容器、判断容器是否为空等等

```java
public class CollectionTest {
    public static void main(String[] args) {
        var strColl = List.of("Java", "Kotlin", "Swift", "Python");
        String[] sa = strColl.toArray(String[]::new);
        System.out.println(Arrays.toString(sa));
    }
}
```

#### 1、使用Lambda表达式遍历集合

```java
public class CollectionEach {
    public static void main(String[] args) {
        //创建一个集合
        HashSet<Object> books = new HashSet<>();
        books.add("轻量级Java EE企业级应用实战");
        books.add("疯狂Java讲义");
        books.add("疯狂Android讲义");
        //遍历forEach()方法遍历集合
        books.forEach(obj -> {
            System.out.println("迭代集合元素：" + obj);
        });
    }
}
```



#### 2、使用Iterator遍历集合元素

Iterator接口也是Java集合框架的成员，但是他与Collection系列、Map系列的集合不一样，：Collection系列集合、Map系列集合主要用于乘装其他对象，而Iterator则主要用于遍历（即迭代访问）Collection中的元素，Iterator对象也称为迭代器。

下面直接上代码看看Iterator遍历元素的例子：

```java
public class IteratorTest {
    public static void main(String[] args) {
        //创建一个集合
        HashSet<Object> books = new HashSet<>();
        books.add("轻量级Java EE企业级应用实战");
        books.add("疯狂Java讲义");
        books.add("疯狂Android讲义");
        //获取books对象的迭代器
        Iterator<Object> it = books.iterator();
        while(it.hasNext()) {
            //强制转换
            String book = (String) it.next();
            System.out.println(book);
            if (book.equals("疯狂Java讲义")) {
                //从集合中删除上一次next()方法返回的元素
                it.remove();
            }
            //为测试变量赋值，不会改变几个元素本身
            book = "测试字符串";
        }
        System.out.println(books);
    }
}
```

> - Iterator必须依附于Collection对象，它本身并不提供乘装对象的能力
> - 使用Iterator对集合元素遍历时，Iterator并非将集合元素本身交给了迭代变量，而是将集合元素的值传给了迭代变量
> - 使用Iterator迭代过程中，一旦发现集合被修改，会抛出异常，代码如下

```java
public class IteratorErrorTest {
    public static void main(String[] args) {
        //创建一个集合
        HashSet<Object> books = new HashSet<>();
        books.add("轻量级Java EE企业级应用实战");
        books.add("疯狂Java讲义");
        books.add("疯狂Android讲义");
        //获取books对象的迭代器
        Iterator<Object> it = books.iterator();
        while(it.hasNext()) {
            //强制转换
            String book = (String) it.next();
            System.out.println(book);
            books.remove(book);
        }
        System.out.println(books);
    }
}
```

Iterator迭代器采用的是快速失败的（fail-fast机制），一旦检测到迭代过程该集合已经被修改，就会抛出` ConcurrentModificationException`异常。



#### 3、使用Lambda表达式遍历Iterator

Java8为Itreater新增了一个` forEachRemaining(Consumer action);`方法，该方法所需的Consumer参数同样也是函数式接口。

代码如下：

```java
public class IteratorEach {
    public static void main(String[] args) {
        //创建一个集合
        HashSet<Object> books = new HashSet<>();
        books.add("轻量级Java EE企业级应用实战");
        books.add("疯狂Java讲义");
        books.add("疯狂Android讲义");
        //获取books对应的迭代器
        Iterator<Object> iterator = books.iterator();
        //使用Lambda表达式来遍历集合元素
        iterator.forEachRemaining(obj -> {
            System.out.println("迭代元素集合：" + obj);
        });
    }
}
```



#### 4、使用foreach循环遍集合元

直接上代码：

```java
```



#### 5、使用Predicate操作集合



#### 6、使用Stream操作集合





### 三、Set集合

#### 1、HashSet类



#### 2、LinkedHashSet类

HashSet有一个子类，LinkedHashSet，LinkHsahSet集合也是根据元素的hashCode值来决定元素的存储位置，但它同时使用链表维护了元素的次序。

当遍历LinkedHashSet集合里的元素时，会按照元素插入的顺序来访问。

```java
```

> 虽然LinkedHashSet使用了链表记录集合元素的添加顺序，但LinkedHashSet依然是HashSet，他不允许元素重复。



#### 3、TreeSet类





#### 4、EnumSet类

#### 5、各Set实现类的性能分析



### 四、List集合

#### 1、改进的List接口和ListIterator

#### 2、ArrayList和Vector实现类

#### 3、固定长度的List



### 五、Queue集合

#### 1、PriorityQueue实现类

#### 2、Deque接口与ArrayDeque实现类

#### 3、LinkedList实现类

#### 4、各种线性表的性能分析



### 六、增强的Map集合

#### 1、Java8为Map新增的方法

#### 2、改进的HashMap和Hashtable实现类

#### 3、LinkedHashMap实现类

#### 4、使用Properties读写属性文件

#### 5、SortedMap接口和TreeMap实现类

#### 6、WeakHashMap实现类



### 七、HashSet和HashMap的性能选择



### 八、操作集合的工具类：Collections

#### 1、排序操作

#### 2、查找、替换操作

#### 3、同步控制

#### 4、设置不可变集合

#### 5、Java9新增的不可变集合



### 九、繁琐的接口：Enumeration



### 十、本章小结









