---
title: Java面向对象
date: 2021-06-04
tags: [对象]
categories: Java
---

### 一、包装类



### 二、处理对象

#### 1、打印对象和` toString()`方法

#### 2、` ==`和` equals`方法



### 三、类成员

#### 1、理解类成员

#### 2、单例（Singleton）类



### 四、final修饰符

#### 1、final成员变量

#### 2、final局部变量

#### 3、final修饰基本类型变量和引用类型变量的区别

#### 4、可执行“宏替换”的final变量

#### 5、final方法

#### 6、final类

#### 7、不可变类

#### 8、缓存实例的不可变类



### 五、抽象类

#### 1、抽象方法和抽象类

#### 2、抽象类的作用



### 六、Java9改进的接口

#### 1、接口的概念

#### 2、Java9中接口的定义

#### 3、接口的继承

#### 4、使用接口

#### 5、接口和抽象类

#### 6、面向接口编程



### 七、内部类

所谓内部类，就是将一个类放在另一个类内部定义，这个定义在其他类内部的类就叫做内部类（嵌套类），包含内部类的类也称为外部类（宿主类）。Java从JDK1.1开始引入内部类，主要有如下作用：

- 内部类提供了更好的封装，可以将内部类隐藏到外部类之内，不允许同包下的其他类访问该类。

  例如：Cow类里面创建一个CowLeg类

- 内部类成员可以直接访问外部类的私有数据

- 匿名内部类适合于创建那些仅需要一次使用的类

此外，使用内部类时还需注意以下两点：

- 内部类比外部类可以多使用三个修饰符：`private`、`protected`、`static`（外部类不可使用）
- 非静态内部类不可拥有静态成员

#### 1、非静态内部类

大多数时候，内部类都被作为成员内部类定义，而不是作为局部内部类。成员内部类是一种与成员变量、方法、构造器和初始化块相似的类成员，局部内部类和匿名内部类不是类成员。

成员内部类分为两种：` 静态内部类`和` 非静态内部类`，使用static修饰的为静态内部类。

经常看到同一个Java源文件里定义了多个类，那不是内部类，他们依然是两个相互独立的类，如下：

```java
//下面两个类相互独立，没有谁是谁的内部类
class A {}
public class B {}
```

> 外部类的上一级程序单元是包，所以只有两个作用域：` 同一个包内`和` 任何位置`，因此只需要两种访问权限：` 包访问权限`和` 公开访问权限`，正好对应于` 省略访问控制符`和` public访问控制符`
>
> 内部类的上一级程序单元是外部类，具有四个作用域：` 同一个类`、` 同一个包`、` 父子类`、` 任何位置`，因此可以使用4种访问控制权限

下面的程序在Cow类里面定义了一个CowLeg非静态内部类，并在方法中直接访问Cow的private的实例变量：

```java
public class Cow {
    private double weight;
    //外部类的构造器
    public Cow(double weight) { this.weight = weight; }
    //定义一个非静态内部类
    private class CowLeg {
        //非静态内部类的两个实例变量
        private double length;
        private String color;
        //内部类的构造器
        public CowLeg(double length, String color) {
            this.length = length;
            this.color = color;
        }
        //非静态内部类的实例方法
        public void info() {
            System.out.println("牛腿颜色为：" + color);
            System.out.println("牛高：" + length);
            //直接访问外部类private修饰的成员变量
            System.out.println("奶牛重：" + weight);
        }
    }
    public void test() {
        var cl = new CowLeg(1.12, "黑白相间");
        cl.info();
    }
    public static void main(String[] args) {
        var cow = new Cow(378.9);
        cow.test();
    }
}
```

编译上面程序，发现文件所在路径下生成了两个class文件，如下图：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/java-object1.png">

成员内部类（包括静态内部类、非静态内部类）的class文件总是以这种形式：`OuterClass$InnerClass.class`

内部类之所以可以访问外部类的private成员，是因为在非静态内部类对象中，保存了一个它所寄生的外部类对象的引用（当调用非静态内部类实例方法时，必须有一个非静态内部类实例，非静态内部类必须寄生在外部类实例里面），如下图所示：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/java-object2.png">

#### 2、静态内部类

#### 3、使用内部类

#### 4、局部内部类

#### 5、匿名内部类



#### 八、Java11增强的Lambda表达式

#### 1、Lambda表达式入门

#### 2、Lambda表达式与函数式接口

#### 3、在Lambda表达式中使用var

#### 4、方法引用与构造器引用

#### 5、Lambda表达式与匿名内部类的联系和区别

#### 6、使用Lambda表达式调用Arrays的类方法



### 九、枚举类

#### 1、手动实现枚举类

#### 2、枚举类入门

#### 3、枚举类成员变量、方法与构造器

#### 4、实现接口的枚举类

#### 5、包含抽象方法的枚举类



### 十、对象与垃圾回收

#### 1、对象在内存中的状态

#### 2、强制垃圾回收

#### 3、finalize方法

#### 4、对象的软、弱和虚引用



### 十一、修饰符的适用范围



### 十二、多版本JAR包

#### 1、jar命令详解

#### 2、创建可执行的JAR包

#### 3、关于JAR包的技巧



### 十三、本章小结





