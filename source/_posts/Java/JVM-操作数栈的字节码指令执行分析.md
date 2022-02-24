

追踪的代码如下：

```java
public class OperandStackTest {
    public void testAddOperation() {
      	//byte short char boolean : 都以int型来保存
        byte i = 15;
        int j = 8;
        int k = i + j;
    }
}
```

之后进行编译，得到字节码文件：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/JVM/classFile.png" align=left width=400>

然后对得到的字节码文件进行反解析：

```shell
javap -verbose OperandStackTest.class
```

也可以直接使用jclasslib插件直接解析：

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/JVM/jclasslib.png" align=left width=400>

该方法对应的字节码指令为：

```text
 0 bipush 15
 2 istore_1
 3 bipush 8
 5 istore_2
 6 iload_1
 7 iload_2
 8 iadd
 9 istore_3
10 return
```

执行引擎将字节码指令翻译成机器指令



我们再添加一些代码，重新编译：

```java
public int getSum() {
  int m = 10;
  int n = 20;
  return m + n;
}

public void testGetSum() {
  //获取上一个栈帧返回的结果，并保存在操作数栈中
  int i = getSum();
  int j = 10;
}
```

看看`testGetSum()`方法反编译结果：

```text
0 aload_0
1 invokevirtual #2 <OperandStackTest.getSum : ()I>
4 istore_1
5 bipush 10
7 istore_2
8 return
```

*aload_0*中的0正是局部变量表里的Slot 0的含义，从局部变量表0的位置加载，也就是`	this`的位置



i++和++i的区别：

```java
public void add() {
  //第一类问题
  int i1 = 10;
  i1++;

  int i2 = 10;
  ++i2;

  //第二类问题
  int i3 = 10;
  int i4 = i3++;

  int i5 = 10;
  int i6 = ++i5;

  //第三类问题
  int i7 = 10;
  i7 = i7++;

  int i8 = 10;
  i8 = ++i8;

  //第四类问题
  int i9 = 10;
  int i10 = i9++ + ++i9;
}
```



