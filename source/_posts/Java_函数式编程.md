---
title: Java函数式编程
date: 2020-12-13
tags: [Java]
categories: Java
---



笔记来源：

[Java疯狂讲义](https://item.jd.com/12518025.html)、[廖雪峰老师的博客](https://www.liaoxuefeng.com/wiki/1252599548343744)

> 对于Java的函数式编程，做一些摘抄以及总结，我是一个快乐的搬运工~~~

### Lambda表达式

#### 一、简介

Lambda表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性，它允许把函数作为一个方法的参数，使代码变的更加简洁紧凑，它允许使用更简洁的代码来创建一个只有一个抽象方法的接口（这种接口被称为函数式接口）的实例。



#### 二、Lambda入门

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



#### 三、Lambda表达式与函数式接口

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



#### 四、方法引用与构造器引用

如果Lambda代码块中只有一条代码，程序就可以省略Lambda中的花括号，甚至还能进行`方法引用`与`构造器引用`



##### 1.引用类方法

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

##### 4.引用构造器

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



#### 五、Lambda表达式与匿名内部类的联系与区别

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



#### 六、使用Lambda表达式调用Arrays的类方法

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

#### 一、简介

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据，英语较好的同学建议直接看[官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)。

那么Stream究竟是啥呢？

我们来康康英文单词`stream`，解释为 `溪流 小溪 数据流`，其实Java中的Stream也是类似于这个的东西。

官方文档的第一句话`A sequence of elements supporting sequential and parallel aggregate operations.`，一个支持顺序和并行聚合的序列，这样说起来这个Stream和List有有啥区别呢？

List存储的每个元素都是已经存储在内存中的某个Java对象，而Stream输出的元素可能并没有预先存储在内存中，而是实时计算出来的。也就是说，如果我没有用到Stream中的元素，它就不存在，相当于Stream只是一个声明的作用。

接下来看一个例子：

```java
int sum = widgets.stream()
    .filter(w -> w.getColor() == RED)
    .mapToInt(w -> w.getWeight())
    .sum();
```

这也是官方文档上的一个例子，`widgets`是一个`Widget`对象的一个`Collection`，我们创建了一个包含`Widget`对象的Stream；`filter`就是一个过滤器，他只留下了` red widgets`；然后`mapToInt`将这条Stream变成了一个`int`的Stream，最后求和。

怎么样？通过这个小例子是不是对Stream有了一点点感觉呢？接下来我们就开始学习使用它吧！



#### 二、Stream的创建方式

- 1.使用Stream.of()静态方法

```java
Stream<String> stream1 = Stream.of("A","B","C","D");
stream1.forEach(System.out::println);  //函数式接口
```

上面的例子传入了可变参数即创建了一个能输出确定元素的Stream。



- 2.基于数组或Collection

```java
Stream<String> stream2 = Arrays.stream(new String[]{"A", "B", "C"});
Stream<String> stream3 = List.of("X","Y").stream();
List<String> list = new ArrayList<String>();
list.add("hello");
list.add("world");
Stream<String> stream4 = list.stream();
```

`stream2`使用`Arrays.stream()`方法把传入的固定数组变成了Stream；

`stream3`是使用`List.of()`生成一个不可变数组，然后通过这个数组创建的流，需要注意的是，`List.of()`是在jdk1.8以后才出现的，所以在jdk1.9版本及以后才能运行；

`stream4`是`list`直接调用`stream()`方法获得的，Stream对于`Collection（List、Set、Queue等）`，直接调用`stream()`方法就可以获得Stream，这种创建Stream的方法都是把一个现有的序列变为Stream，它的元素是固定的。



- 基于Supplier

```java
public class StreamTest {
    public static void main(String[] args) {
        Stream<Integer> stream5 = Stream.generate(new NatualSupplier());
        // 无限序列必须先变成有限序列再打印
        stream5.limit(10).forEach(System.out::println);
    }
    static class NatualSupplier implements Supplier<Integer> {
        private int n=0;
        @Override
        public Integer get() {
            n++;
            return n;
        }
    }
}
```

`stream5`通过`Stream.generate()`方法创建的Stream，它需要传入一个Supplier对象，基于`Supplier`创建的`Stream`会不断调用`Supplier.get()`方法来不断产生下一个元素，这种`Stream`保存的不是元素，而是算法，它可以用来表示无限序列。

上述代码我们用一个`Supplier<Integer>`模拟了一个无限序列（当然受`int`范围限制不是真的无限大）。如果用`List`表示，即便在`int`范围内，也会占用巨大的内存，而`Stream`几乎不占用空间，因为每个元素都是实时计算出来的，用的时候再算。



- 其他方式

通过一些API提供的接口创建Stream

例如，Files类的lines()方法可以把一个文件变成一个Stream，每个元素代表文件的一行内容：

```java
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) {}
```



另外，正则表达式的Pattern对象有一个`splitAsStream()`方法，可以直接把一个长字符串分割成Stream序列而不是数组：

```java
Pattern p = Pattern.compile("\\s+");
Stream<String> s = p.splitAsStream("The quick brown fox jumps over the lazy dog");
s.forEach(System.out::println);
```



#### 三、Stream的几种方法

##### 1.Stream.map()

`Stream.map()`是Stream最常用的一个转换方法，它把一个Stream转换为另一个Stream；所谓map操作，就是把一种操作运算，把一个序列映射到另外一个序列的每一个元素上。

例如下面的一个例子，对x计算它的平方，可以使用函数`f(x) = x * x`，我们把这个函数映射到一个序列`1，2，3，4，5`上，就得到了另一个序列`1，4，9，16`：

```java
Stream<Integer> s1 = Stream.of(1,2,3,4);
Stream<Integer> s2 = s1.map(n -> n*n);
```

如果我们点进去查看Stream的源码：`<R> Stream<R> map(Function<? super T, ? extends R> mapper);`发现map()方法接收的对象是Function接口对象，而Function接口对象呢，它定义了一个`apply()`方法，负责把一个`T`类型转换成`R`类型：`R apply(T t);`，由此可见`Stream.map()`的功能。

下面我们看看他的简单应用：

```java
public class Main {
    public static void main(String[] args) {
        List.of("  Apple ", " pear ", " ORANGE", " BaNaNa ")
                .stream()
                .map(String::trim) // 去空格
                .map(String::toLowerCase) // 变小写
                .forEach(System.out::println); // 打印
    }
}
```



##### 2.Stream.filter()

`Stream.filter()`从字面也能理解，即过滤器。`filter()`方法原码：`Stream<T> filter(Predicate<? super T> predicate);`它定义了一个test()方法，再深入：`boolean test(T t);`可以看到这个方法负责判断元素是否符合条件。

```java
public class mapTest {
    public static void main(String[] args) {
        Stream.generate(new LocalDateSupplier())
                .limit(30)
                .filter(Idt -> Idt.getDayOfWeek() == DayOfWeek.SUNDAY || Idt.getDayOfWeek() == DayOfWeek.SATURDAY)
                .forEach(System.out::println);
    }

    static class LocalDateSupplier implements Supplier<LocalDate> {
        LocalDate start = LocalDate.of(2020, 1,1);
        int n = -1;
        public LocalDate get() {
            n++;
            return start.plusDays(n);
        }
    }
}
```

可以看到，`filter()`除了常用于数值外，也可应用于任何Java对象，上面的例子就是从一组给定的`LocalDate`中过滤掉工作日，以便得到休息日。



##### 3.Stream.reduce()

`map()`和`filter()`都是Stream的转换方法，而`Stream.reduce()`则是Stream的一个聚合方法，它可以把一个Stream的所有元素按照聚合函数聚合成一个结果，下面看一个例子：

```java
public class Main {
    public static void main(String[] args) {
        int sum = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (acc, n) -> acc + n);
        System.out.println(sum);
    }
}
```

咱们依然是来先看看`reduce()`的源码：`T reduce(T identity, BinaryOperator<T> accumulator);`  可以看到`reduce()`方法传入的一个对象是`BinaryOperator`接口，继续前进，发现`BinaryOperator`接口继承了`BiFunction<T,T,T>`：`public interface BinaryOperator<T> extends BiFunction<T,T,T>`  最后终于在`BiFunction`中找到了`R apply(T t, U u);`这么一个方法，`apply()`方负责把上次累加的结果和本次的元素 进行运算，并返回累加的结果。

聊了这么多，最终的结果就是我们在`reduce()`方法中传入的Lambda表达式`(acc, n) -> acc + n`实现了`R apply(T t, U u);`，并将聚合的结果给了acc返回，而传入的第一个参数0的作用是初始化结果为0。



除了求和，还有求积：

```java
public class Main {
    public static void main(String[] args) {
        int s = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(1, (acc, n) -> acc * n);
        System.out.println(s);
    }
}
```



除了可以对数值进行累积计算外，灵活运用`reduce()`也可以对Java对象进行操作。下面的代码演示了如何将配置文件的每一行配置通过`map()`和`reduce()`操作聚合成一个`Map<String, String>`：

```java
public class Main {
    public static void main(String[] args) {
        // 按行读取配置文件:
        List<String> props = List.of("profile=native", "debug=true", "logging=warn", "interval=500");
        Map<String, String> map = props.stream()
                // 把k=v转换为Map[k]=v:
                .map(kv -> {
                    String[] ss = kv.split("\\=", 2);
                    return Map.of(ss[0], ss[1]);
                })
                // 把所有单个Map对象聚合到一个Map集合:
                .reduce(new HashMap<String, String>(), (m, kv) -> {
                    m.putAll(kv);
                    return m;
                });
        // 打印结果:
        map.forEach((k, v) -> {
            System.out.println(k + " = " + v);
        });
    }
}
```

我们来看看上面的程序，理一下思路：

首先，我们先创建一个`String`类型的`List`即`props`，`props.stream()`将`props`变成一条Stream，然后在`.map()`中将Stream中的每一个String对象都进行了分割，两段字符串分别存在`ss[0], ss[1]`中，并调用` Map.of(ss[0], ss[1])`将每一个对象都变成了一个Map对象，最后在`.reduce()`方法中，初始值为一个`HashMap<String, String>()`对象，将Stream中的每一个Map对象放到`HashMap<String, String>()`集合中并返回。

费了好一番口舌，还是感觉原理有一点难理解，建议多看看源码。



 #### 四、将Stream输出为集合

##### 0.前言

 前面介绍了Stream的几个常见操作：map()、filter()、reduce()
 这些操作对Stream来说可以分为两类，

* 一类是转换操作，即把一个Stream转换为另一个Stream 例如`map()`和`filter()`
* 另一类是聚合操作，即对Stream的每个元素进行计算，得到一个确定的结果，例如`reduce()`

我们要知道对于`Stream`来说，对其进行`转换操作`并不会触发任何计算！如下

```java
public class Main {
    public static void main(String[] args)     {
        Stream<Long> s1 = Stream.generate(new NatualSupplier());
        Stream<Long> s2 = s1.map(n -> n * n);
        Stream<Long> s3 = s2.map(n -> n - 1);
        System.out.println(s3); // java.util.stream.ReferencePipeline$3@49476842
    }
}

class NatualSupplier implements Supplier<Long> {
    long n = 0;
    public Long get() {
        n++;
        return n;
    }
}
```



而`聚合操作`则不一样，`聚合操作`会立刻促使`Stream`输出它的每一个元素，并依次纳入计算，以获得最终结果，如下

```java
Stream<Long> s1 = Stream.generate(new NatualSupplier());
Stream<Long> s2 = s1.map(n -> n * n);
Stream<Long> s3 = s2.map(n -> n - 1);
Stream<Long> s4 = s3.limit(10);
s4.reduce(0, (acc, n) -> acc + n);
```

如果我们将`Stream<Long> s4 = s3.limit(10);`注释掉，就会发生错误，其原因就是`聚合操作`会即时计算。

可见，`聚合操作`是真正需要从`Stream`请求数据的，对一个`Stream`做聚合计算后，结果就不是一个`Stream`，而是一个其他的Java对象。



##### 1.输出为List

因为List的元素是确定的Java对象，因此，把Stream变为List不是一个`转换操作`，而是一个`聚合操作`，它会强制Stream输出每个元素。

```java
Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
List<String> list = stream.filter(s -> s != null && !s.isBlank()).collect(Collectors.toList());
System.out.println(list);
```

把Stream的每个元素收集到`List`的方法是调用`collect()`并传入`Collectors.toList()`对象，它实际上是一个`Collector`实例，通过类似`reduce()`的操作，把每个元素添加到一个收集器中（实际上是`ArrayList`），类似的，`collect(Collectors.toSet())`可以把Stream的每个元素收集到`Set`中。



##### 2.输出为数组

把Stream的元素输出为数组和输出为List类似，我们只需要调用`toArray()`方法，并传入数组的“构造方法”：

```java
List<String> stringList = List.of("Apple", "Banana", "Orange");
String[] array = list.stream().toArray(String[]::new);
```



##### 3.输出为Map

如果我们要把Stream的元素收集到`Map`中，可以使用之前的先转换再聚合的方法

```java
Stream<String> streamm = Stream.of("APPL:apple", "MSFT:Microsoft");
Map<String, String> map = streamm.map( str -> {
    String[] ss = str.split(":",2);
    return Map.of(ss[0], ss[1]);
}).reduce( new HashMap<String, String>(), (m, kv)->{
    m.putAll(kv);
    return m;
});
```



当然我们还有其他方法，我们可以指定两个映射函数，分别把元素映射为`key`和`value`：

```java
streamm = Stream.of("APPL:apple", "MSFT:Microsoft");
Map<String, String> mapp = streamm.collect(Collectors.toMap(
    // 将元素s映射为key
    s -> s.substring(0, s.indexOf(":")),
    // 将元素v映射为value
    v -> v.substring(v.indexOf(":")+1)));
System.out.println(mapp);
```



##### 4.分组输出

Stream还有一个强大的分组功能，可以按组输出。我们看下面的例子：

```java
List<String> Fruit = List.of("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
Map<String, List<String>> groups = Fruit.stream()
    .collect(Collectors.groupingBy( s -> s.substring(0,1), Collectors.toList() ));
System.out.println(groups);
```

分组输出使用`Collectors.groupingBy()`，它需要提供两个函数：`分组的key`与`分组的value`

`分组的key`这里使用`s -> s.substring(0, 1)`，表示只要首字母相同的String分到一组

`分组的value`这里直接使用`Collectors.toList()`，表示输出为List

假设有这样一个Student类，包含学生姓名、班级和成绩：

```java
class Student {
    int classId; // 班级
    String name; // 名字
}
```

如果我们有一个Stream<Student>，利用分组输出，可以非常简单地按班级把Student归类

如下面的例子：

```java
public class test {
    static class Student {
        int classId; // 班级
        String name; // 名字

        public Student(int classId, String name) {
            this.classId = classId;
            this.name = name;
        }

        public int getClassId() {
            return classId;
        }

        public void setClassId(int classId) {
            this.classId = classId;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

    public static void main(String[] args) {
        List<Student> students = new  ArrayList<Student>();
        students.add(new Student(1, "xiaoming"));
        students.add(new Student(2, "xiaohong"));
        students.add(new Student(1, "panghu"));
        Map<Integer, List<Student>> studentsByClass = students.stream()
                .collect(Collectors.groupingBy(s -> s.getClassId(), Collectors.toList()));
        System.out.println(studentsByClass);
    }
}
```



#### 五、其他操作

`Stream`提供的操作分为两类：`转换操作`和`聚合操作`。除了前面介绍的常用操作外，`Stream`还提供了一系列非常有用的方法。



##### 1.排序

对Stream的元素进行排序十分简单，只需调用`sorted()`方法，此方法要求Stream的每个元素必须实现`Comparable`接口。如果要自定义排序，传入指定的`Comparator`即可：

```java
List<String> list = List.of("hello", "xty", "xwy", "jhl", "aaa")
	.stream()
	.sorted( (s1, s2) -> {
		return -s1.compareTo(s2);
	} )
	.collect(Collectors.toList());
System.out.println(list);
```

注意`sorted()`只是一个转换操作，它会返回一个新的Stream，所以我们后来要`.collect(Collectors.toList());`把它变成一个`List`



##### 2.去重

对一个Stream的元素进行去重，没必要先转换为`Set`，可以直接用`distinct()`：

```java
List<String> liss = List.of("A", "B", "A", "C", "B", "D").stream().distinct().collect(Collectors.toList());
System.out.println(liss);
```



##### 3.截取

截取操作也是一个转换操作，将返回新的Stream：

```java
liss = List.of("A", "B", "C", "D", "E", "F")
    .stream()
    .skip(2) // 跳过A, B
    .limit(3) // 截取C, D, E
    .collect(Collectors.toList());
```



##### 4.合并

将两个Stream合并为一个Stream可以使用Stream的静态方法`concat()`：

```java
Stream<String> s1 = List.of("A", "B", "C").stream();
Stream<String> s2 = List.of("D", "E").stream(); // 合并:
Stream<String> s = Stream.concat(s1, s2);
System.out.println(s.collect(Collectors.toList())); // [A, B, C, D, E]
```



##### 5.flatMap

如果`Stream`的元素是集合：

```java
Stream<List<Integer>> s = Stream.of(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9));
```

而我们希望把上述`Stream`转换为`Stream<Integer>`，就可以使用`flatMap()`：

```java
Stream<Integer> i = s.flatMap(list -> list.stream());
```



##### 6.并行

通常情况下，对`Stream`的元素进行处理是单线程的，即一个一个元素进行处理。但是很多时候，我们希望可以并行处理`Stream`的元素，因为在元素数量非常大的情况，并行处理可以大大加快处理速度。

把一个普通`Stream`转换为可以并行处理的`Stream`非常简单，只需要用`parallel()`进行转换：

```java
Stream<String> s = ...
String[] result = s.parallel() // 变成一个可以并行处理的Stream
    .sorted() // 可以进行并行排序
    .toArray(String[]::new);
```

经过`parallel()`转换后的`Stream`只要可能，就会对后续操作进行并行处理。我们不需要编写任何多线程代码就可以享受到并行处理带来的执行效率的提升。



##### 7.其他聚合方法

除了`reduce()`和`collect()`外，`Stream`还有一些常用的聚合方法：

- `count()`：用于返回元素个数；
- `max(Comparator<? super T> cp)`：找出最大元素；
- `min(Comparator<? super T> cp)`：找出最小元素。

针对`IntStream`、`LongStream`和`DoubleStream`，还额外提供了以下聚合方法：

- `sum()`：对所有元素求和；
- `average()`：对所有元素求平均数。

还有一些方法，用来测试`Stream`的元素是否满足以下条件：

- `boolean allMatch(Predicate<? super T>)`：测试是否所有元素均满足测试条件；
- `boolean anyMatch(Predicate<? super T>)`：测试是否至少有一个元素满足测试条件。



#### 五、Stream用法小结

`Stream`提供的常用操作有：

转换操作：`map()`，`filter()`，`sorted()`，`distinct()`；

合并操作：`concat()`，`flatMap()`；

并行处理：`parallel()`；

聚合操作：`reduce()`，`collect()`，`count()`，`max()`，`min()`，`sum()`，`average()`；

其他操作：`allMatch()`, `anyMatch()`, `forEach()`