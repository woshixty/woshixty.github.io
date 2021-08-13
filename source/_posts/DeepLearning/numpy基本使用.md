---
title: NumPy常用操作
date: 2021-04-18
tags: [deep-learning]
categories: 深度学习
---

# NumPy常用操作

## 一、NumPy是什么

NumPy是Python的一个用于科学计算的基础包。它提供了多维数组对象，多种衍生的对象（例如隐藏数组和矩阵）和一个用于数组快速运算的混合的程序，包括数学，逻辑，排序，选择，I/O，离散傅立叶变换，基础线性代数，基础统计操作，随机模拟等等。--摘自知乎

NumPy提供了两种基本的对象:` ndarray`和` ufunc`. ndarray是存储单一数据类型的多维数组, 而ufunc则是对数组处理的函数

## 二、NumPy主要特点

- ndarray, 快速, 节省空间的多维数组, 提供数组化的算术运算和高级的广播功能
- 使用标准的数学函数对整个数组的数据进行快速运算, 不需要写循环
- 读/写磁盘上的阵列数据和操作存储器映像文件的工具
- 线性代数、随机数生成、傅立叶变换
- 集成C、C++、Fortran代码的工具

## 三、NumPy的使用

### 1、生成NumPy的ndarray的几种方式

NumPy封装了一种新的数据类型: ` ndarray`, 一个多维数组对象, 该对象封装了许多常用的数学运算符号。

#### a、从已有的数据中创建

- 将列表转化为ndarray. (可以尝试着把两个东西输出看看, 会有不一样的发现)

```python
num_list = [3.14, 1, 3, 92]
nd = np.array(num_list)
```

- 前套列表转化为多位ndarray

```python
num_list = [[3.14, 1, 3, 92], [1, 2, 3, 4]]
nd = np.array(num_list)
```

#### b、使用random模块生成ndarray

有时候我们需要对一些变量进行初始化, 我们就可以使用random模块来生成。random又分为多种函数: 1.random生成0~1之间的随机数、2.uniform生成均匀分布的随机数、3.randn生成标准正态的随机数、4.normal生成正态分布的、5.shuffle随机打乱顺序、6.seeb设置种子. 如下代码:

```python
# 你每次执行都是不一样的结果
nd = np.random.random([3, 3])
print(nd)
```

我的打印结果:

```shell
[[0.0818346  0.23860994 0.64879857]
 [0.06154791 0.72195076 0.9455963 ]
 [0.44779405 0.17841636 0.98912378]]
```

#### c、创建特定形状的多维数组

可以生成一些特殊矩阵, 直接上代码:

```python
# 3×3矩阵，矩阵元素均为0
nd1 = np.zeros([3, 3])
print(nd1)

# 3×3矩阵，矩阵元素均为1
nd2 = np.ones([3, 3])
print(nd2)

# 3阶的单位矩阵
nd3 = np.eye(3)
print(nd3)

# 3阶对角矩阵
nd4 = np.diag([1, 2, 3])
print(nd4)
```

将数据保存到磁盘然后读取

```python
nd1 = np.random.random([3, 3])
print(nd1)
numpy.savetxt(X=nd1, fname='test.txt')
nd0 = np.loadtxt('test.txt')
print(nd0)
```

#### d、利用arange函数

arange是numpy模块中的函数, 废话少说, 实践出真知:

```python
# 从0到10，每次加一生成的数组
print(np.arange(10))
# 从3到10，每次加一生成的数组
print(np.arange(3, 10))
# 从1到11（不等于11），每次加二生成的数组
print(np.arange(1, 11, 2))
# 从9一直到-1（不等于-1），每次减一生成的数组
print(np.arange(9, -1, -1))
```

可以看见, 该函数格式为: ` arange([start], stop, [step], dtype=None)`, start和stop指定范围, start默认为0, stop必须指定, step为步长。



### 2、NumPy存取元素

```python
nd = np.diag([1, 2, 3, 4, 5, 6, 7, 8])
print(nd)

# 获取第四个元素，一个元素即一个数组，起始元素为0
print("nd[4]:")
print(nd[4])

# 截取一段数据，从3开始，小于6
print("nd[3:6]:")
print(nd[3:6])

# 截取固定间隔数据，从3开始，小于6，每次加二
print("nd[1:6:2]:")
print(nd[1:6:2])

# 倒序取数
print("nd[::-1]:")
print(nd[::-1])
```

以上操作仅仅是操作矩阵的行, 如要操作列, 继续往下

```python
nd = np.arange(25).reshape([5, 5])
print(nd)

# 截取多维数组一个区域内的数据，如下：截取2到3行，2到四列的数据
print("nd[1:3, 1:2]:")
print(nd[1:3, 1:4])

# 截取多维数组一个区域内的数据，生成一个一维数组
print("nd[(nd > 3) & (nd < 10)]:")
print(nd[(nd > 3) & (nd < 10)])

# 截取多维数组的2、3行
print("nd[1:3, :]:")
print(nd[1:3, :])

# 截取多维数组的2、3列
print("nd[:, 1:3]:")
print(nd[:, 1:3])
```

以上学会了吗?

另外, 还可以通过random.choice函数从指定的样本随机抽取数据, 此处略。

### 3、矩阵操作

```python
nd = np.arange(9).reshape([3, 3])
print(nd)

# 矩阵砖置
print("nd矩阵转置：")
print(np.transpose(nd))

# 矩阵的乘法
print("a✖️b矩阵的乘法：")
a = np.arange(12).reshape([3, 4])  # 三行四列的矩阵
b = np.arange(8).reshape([4, 2])  # 四行两列的矩阵
print(a.dot(b))

# 求矩阵的迹
print("a矩阵的迹：")
print(a.trace())

# 计算行列式
print("计算c的行列式：")
c = np.arange(4).reshape([2, 2])
print(np.linalg.det(c))

# 计算逆矩阵
print("c的逆矩阵：")
print(np.linalg.solve(c, np.eye(2)))
```

我们在上面使用了` numpy.linalg`的函数, 下面咱们整理了一份表(` numpy.linalg`常用的函数):

| 函数  | 说明                               |
| ----- | ---------------------------------- |
| diag  | 一维数组的形式返回方阵的对角线元素 |
| dot   | 矩阵乘法                           |
| trace | 求迹, 对角线元素之和               |
| det   | 计算矩阵列式                       |
| eig   | 计算方阵的特征值与特征向量         |
| inv   | 计算方阵的逆                       |
| qr    | 计算qr分解                         |
| svd   | 计算奇异值分解svd                  |
| solve | 解线性方程组: Ax=b, 其中A为方阵    |
| lstsq | 计算Ax=b的最小二乘解               |

### 4、数据的合并与展开

#### a、合并一维数组

#### b、多维数组的合并

#### c、矩阵展平

### 5、通用函数



















