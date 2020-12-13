---
title: Java函数式编程
date: 2020-12-13
tags: [Java]
categories: Java
---

### Lambda表达式

#### 简介

Lambda表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性，它允许把函数作为一个方法的参数，使代码变的更加简洁紧凑，它允许使用更简洁的代码来创建一个只有一个抽象方法的接口（这种接口被称为函数式接口）的实例。



#### Lambda入门

下面用一个小例子来简单说明

以`Comparator`为例，我们想要调用`Arrays.sort()`时，可以传入一个`Comparator`实例，以匿名类方式编写如下

```java
String[] strings = new String[] {"Apple", "Orange", "Banana", "Lemon"};
Arrays.sort(strings, new Comparator<String>() {
    public int compare(String o1, String o2) {
    return o1.compareTo(o2);
	}
});
```

从Java 8开始，我们可以用Lambda表达式替换 <u>单方法</u> 接口（注意是单方法！！！）如下

```java
Arrays.sort(strings, (s1, s2) -> {
	return s1.compareTo(s2);
});
```

其中，参数是`(s1, s2)`，参数类型可以省略，因为编译器可以自动推断出`String`类型。`-> { ... }`表示方法体，所有代码写在内部即可，返回值的类型也是由编译器自动推断的，这里推断出的返回值是`int`，因此，只要返回`int`，编译器就不会报错。

从上面的例子可以看出：Lambda的代码块会代替实现抽象方法的方法体，Lambda就相当于一个匿名的方法。



#### Lambda表达式与函数式接口

Lambda表达式的类型，也被称为`目标类型`（target type），Lambda的目标类型必须是`函数式接口`（functional interface）。所谓函数式接口就是<u>只包含一个抽象方法的接口</u>（但是可以包含多个默认方法，类方法，但是只能声明一个抽象方法）。例如 `Runnable`、`ActionListener`都是函数式接口。

> Java8专门为函数式接口提供了`@FunctionalInterface`注解，它用于告诉编译器严格检查--检查该接口必须是函数式接口，否则编译器报错。

由于Lambda表达式的结果就是被当成对象，因此可以直接进行赋值如下

```java
Runnable runnable = () -> {
    for (int i=0; i<100; i++)
    	System.out.println(i);
};
```

`Runnable`接口只包含一个无参数的方法，Lambda实现了该接口中`唯一的`、`无参数`的方法，因此就实现了一个`Runnable`对象

> `Runnable`是Java本身提供的一个函数式接口

从以上分析可知Lambda表达式实现的是匿名方法，它有以下两个限制：

- 目标类型必须是明确的函数式接口
- 只能为函数式接口创建对象，Lambda只能实现一个方法他只能为<u>只有一个抽象方法的接口</u>（函数式接口）创建对象



#### 方法引用与构造器引用

##### 1.引用类方法

如果Lambda代码块中只有一条代码，程序就可以省略Lambda中的花括号，甚至还能进行`方法引用`与`构造器引用`

##### 引用类方法

先直接上代码：

```java
public class Lambda {
    @FunctionalInterface
    interface Convert{
        Integer convert(String form);
    }
    
    public static void main(String[] args) {
        //下面的代码使用Lambda创建了Convert对象
        Convert convert = form -> Integer.valueOf(form);
    }
}
```

函数式包含一个`convert`的抽象方法，将String转成Integer，由于表达式所实现的`convert()`需要返回值，所以Lambda把这条代码作为返回值。上面的代码块只有一行调用类方法的代码，因此可以使用如下方法进行替换：

```java
Convert convert = Integer::valueOf;
```

调用`Integer`类方法中的`valueOf()`方法来实现Convert接口的唯一的抽象方法。

##### 2.引用特定对象的实例方法

看看下面的示例：

```java
Convert convert = form -> "fkit.org".indexOf(form);
```

` convert()`方法执行体就是Lambda表达式的代码块部分，下面请看方法引用代替Lambda,引用特定对象的实例方法：

```java
Convert convert = "fkit.org"::indexOf;
```

##### 3.引用某类对象的实例方法

下面直接先看看代码：

```java
public class Lambda {

    @FunctionalInterface
    interface MyTest{
        String test(String a, int b, int c);
    }

    public static void main(String[] args) {
        MyTest mt = (a,b,c) -> a.substring(b, c);
        String str = mt.test("abcdefghijk", 2, 9);
        System.out.println(str);
    }
}
```

上面的Lambda的代码块只有`a.substring(b, c)`，使用了`a`这个String对象的`substring()`方法，当然下面的代码也能实现：

```java
MyTest myTest = String::substring;
```

对于上面这个实例的方法引用，方法引用代替Lambda表达式，引用某类的某类对象的实例方法，<u>函数式接口中被实现方法的第一个参数作为调用者，后面的参数全部传给该方法作为参数</u>。

##### 5.引用构造器

下面看构造器引用：

```java
public class Lambda {

    @FunctionalInterface
    interface MyTest{
        JFrame win(String title);
    }

    public static void main(String[] args) {
        MyTest myTest = (String a) -> new JFrame(a);
        JFrame jFrame = myTest.win("My Window");
        System.out.println(jFrame);
    }
}
```

上面代码调用`myTest`对象的`win()`方法时，`myTest`对象就是Lambda表达式创建的，因此代码块的执行体就是`new JFrame(a)`并将这条语句的返回值作为方法的返回值。同样的，我们也可以用下面的代码去替换：

```java
MyTest myTest = JFrame::new;
```

上面这种构造器引用代替Lambda的做法，函数式接口中被实现方法的全部参数传给该构造器作为参数。



#### Lambda表达式与匿名内部类的联系与区别

从上面的种种实例可以看出`Lambda表达式`是`匿名内部类`的一种简化。

##### 共同点

- 都可以直接访问`effectively finally`的局部变量，以及外部类的成员变量（包括实例变量与类变量）
- 都可以调用从接口中继承的默认方法

下面是喜闻乐见的演示环节：

```java
public class Lambda {
    @FunctionalInterface
    interface Displayable{
        void display();  //抽象方法
        default int add(int a, int b) {  //默认方法
            return a+b;
        }
    }

    private int age=12;
    private static String name="Java";

    public void test() {
        String blog = "XTY' Blog";
        Displayable displayable = () -> {

            //访问`effectively finally`的局部变量
            System.out.println(blog);

            //访问外部类的实例变量和类变量
            System.out.println(age);  //注意！当我在执行函数中（即public static void main）添加这个语句时，会报错。
            System.out.println(name);
        };
        displayable.display();
        displayable.add(1,2);
    }
}
```



##### 不同点

- 匿名内部类可以为任意接口创建实例，不管有多少个抽象方法，但是Lambda表达式只能为函数接口创建实例。
- 匿名内部类可以为抽象类甚至普通类创建实例，但是Lambda表达式，依然只能是函数式接口。
- 匿名内部类允许在实现的抽象方法的方法体内调用接口中定义的默认方法，而Lambda依然不可以。



#### 使用Lambda表达式调用Arrays的类方法

Arrays类有些方法需要实现Comparator、XxxOperation、XxxFunction等接口的实例，这些都是函数式接口，下面演示一个：

```java
public static void main(String[] args) {
    String[] language = new String[] {"java", "c++", "go", "python", "c"};
    Arrays.sort(language, (s1, s2) -> s1.compareTo(s2));
    System.out.println(String.join(",", language));
}
```



至此，Lambda算是简单入了一个门啦！！！



### Stream





``