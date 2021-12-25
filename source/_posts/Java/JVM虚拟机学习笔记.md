## Java虚拟机栈

### 1、Java虚拟机栈帧是什么？

Java虚拟机栈（Java Virtual Machine Stack），早期也叫Java栈，其内部保存着一个一个栈帧（Stack Frame），对应着一次一次的Java方法调用，为线程私有

### 2、Java虚拟机生命周期

生命周期与线程一致

### 3、Java虚拟机的作用

保存局部变量、部分结果，并参与方法的调用与返回

> 局部变量 VS 成员变量
>
> 基本数据变量 VS 引用类型变量（类、数组、接口）

### 4、Java虚拟机的特点

（1）快速有效的分配存储方式

（2）JVM对Java栈的操作只有两个

> 每个方法执行，伴随着进栈（入栈、压栈）
>
> 执行结束后的出栈工作

（3）对于栈来说不存在垃圾回收的问题

>  GC、OOM

### 3、栈中可能出现的问题

StackOverflowError、OutOfMemoryError



