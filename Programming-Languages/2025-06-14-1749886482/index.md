

# C++ 实现动态代理（不定参数模板使用方法）

下面是采用C++实现的一个动态代理的案例：  
背景：通过代理类Proxy将Base的派生类进行一个代理，比如，不满18周岁，就不能做里面的dowork重载函数

```cpp
#ifndef PROXY_H
#define PROXY_H

#include <functional>
#include <iostream>
#include <memory>
#include <tuple>
//这是基类
class Base {
 public:
  using BasePtr = std::shared_ptr<Base>;
  Base(int age);
  virtual ~Base() = default;

  int age() const { return m_age; }

//这是两个接口
  virtual void dowork(int cnt) = 0;
  virtual void dowork(std::string &cnt, int other) = 0;

 private:
  int m_age{0};
};
using BasePtr = std::shared_ptr<Base>;

//这是派生类，实现其中的重载函数
class NetAccess : public Base {
 public:
  using NetAccessPtr = std::shared_ptr<NetAccess>;
  NetAccess(int age);

  virtual void dowork(int cnt) final;
  virtual void dowork(std::string &cnt, int other) final;
};
using NetAccessPtr = std::shared_ptr<NetAccess>;

//代理类，它包含了一个被代理类的成员指针，然后通过不定参数模板执行函数来传参，确定代理类的重载函数。
class Proxy {
 public:
  using ProxyPtr = std::shared_ptr<Proxy>;
  Proxy(const BasePtr p);

//不定参数模板函数
  template <class... args>
  void doSomething(args &&... paras) {//在模板类中&& 表示万能引用，既可以是左值，又可以是右值
  //这里做个过滤，一般来说代理的话，主要用来做过滤使用
    if (m_people->age() >= 18)
    //通过不定参数来确定重载函数，另外通过万能引用来使用完美转发
      m_people->dowork(std::forward(paras)...);
    else {
      std::cout << "age is < 18, so you cannot access net" << std::endl;
    }
  }

 private:
  BasePtr m_people;
};
using ProxyPtr = std::shared_ptr<Proxy>;
#endif  // PROXY_H
```

cpp

```cpp
#include "proxy.h"
#include <iostream>

Base::Base(int age) : m_age(age) {}

NetAccess::NetAccess(int age) : Base(age) {}

void NetAccess::dowork(int cnt) {
  std::cout << "one data:" << cnt << std::endl;
}

void NetAccess::dowork(std::string &cnt, int other) {
  std::cout << "two data:" << cnt << "," << other << std::endl;
}

Proxy::Proxy(const BasePtr p) : m_people(p) {}
```

std::forward会将输入的参数原封不动地传递到下一个函数中，这个“原封不动”指的是，如果输入的参数是左值，那么传递给下一个函数的参数的也是左值；如果输入的参数是右值，那么传递给下一个函数的参数的也是右值

> 插一句，类的[拷贝构造](https://so.csdn.net/so/search?q=%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0&spm=1001.2101.3001.7020)一定要使用引用，否则会一直递归循环下去。

下面是具体用法

```cpp
int main() {
  NetAccessPtr net = std::make_shared<NetAccess>(19);
  ProxyPtr proxy = std::make_shared<Proxy>(net);
  int a = 110;
  std::string h="hello";
  proxy->doSomething(a);
   proxy->doSomething(h,a);
}
```

但是这个代理模式还是有点问题，必须有很多重载函数，比如添加一个新的不同参数的方法，就要重载一个同名函数，多少有点违背了对扩展开放，对修改封闭。  
我们能不能写一个代理类，可以接收任意类型参数的对象成员函数呢，答案是可以的：

```cpp
template <class T, class F, class... args>
class Proxy2 {
 public:
  Proxy2(T *p, F (T::*f)(args...)) : m_people(p), m_f(f) {}

  void doSomething(args &&... paras) {
    if (m_people->age() >= 18){
    //特别注意这里的调用m_f前面有个*
      (m_people->*m_f)(std::forward<args>(paras)...);
    }else {
      std::cout << "age is < 18, so you cannot access net" << std::endl;
    }
  }

 private:
  T *m_people;
  F (T::*m_f)(args...);
};

//这是个工厂模板函数
template <class T, class F, class... args>
Proxy2<T, F, args...> create(T *t, F (T::*f)(args...)) {
  return Proxy2<T, F, args...>(t, f);
}

int main() {
  NetAccessPtr net = createPtr<NetAccess>(19);  
  std::string name = "hello";
  std::string name2 = "word";
  auto proxy = create(net.get(), &NetAccess::buy);
  proxy.doSomething(name, name2);
}


```

上面是裸指针，下面是[智能指针](https://so.csdn.net/so/search?q=%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88&spm=1001.2101.3001.7020)

```cpp
template <class T, class F, class... args>
class Proxy2 {
 public:
  Proxy2(std::shared_ptr<T> p, F (T::*f)(args...)) : m_people(p), m_f(f) {}

  void doSomething(args &&... paras) {
    if (m_people->age() >= 18) {
      (m_people.get()->*m_f)(std::forward<args>(paras)...);
    } else {
      std::cout << "age is < 18, so you cannot access net" << std::endl;
    }
  }

 private:
  std::shared_ptr<T> m_people;
  F (T::*m_f)(args...);
};

//这个工厂函数必须放在外边，如果放在类里面（static形式）会有问题
template <class T, class F, class... args>
std::shared_ptr<Proxy2<T, F, args...>> create(std::shared_ptr<T> t,
                                              F (T::*f)(args...)) {
  return std::make_shared<Proxy2<T, F, args...>>(t, f);
}

int main() {
  NetAccessPtr net = createPtr<NetAccess>(19);  
  std::string name = "hello";
  std::string name2 = "word";
  auto proxy = create(net, &NetAccess::buy);
  proxy->doSomething(name, name2);
}
```

上面就是本次博客主要内容，通过不定参数模板实现了两个不同规格的代理类，用到了不定参数，完美转发。

下面是一个泛化的工厂类，来创建智能指针

```cpp
template <class T, class... args>
std::shared_ptr<T> createPtr(args&&... params) {
  return std::make_shared<T>(std::forward<args>(params)...);
}
```

具体使用，还是按照上述类来讲解：

```cpp
int main() {
//我们发现我们只要指定第一个参数T的类型就可以了
  NetAccessPtr net =
      createPtr<NetAccess>(19);            // std::make_shared<NetAccess>(19);
  ProxyPtr proxy = createPtr<Proxy>(net);  // std::make_shared<Proxy>(net);
  int a = 110;

  proxy->doSomething("hello", a);
}
```

关于不定参数模板函数还有一个递归特性

```cpp
//递归终止条件
void my_print()
 { std::cout << "end" << std::endl;
 return;
  }
  
template <class T, class... args>
void my_print(T para, args... paras) {
  std::cout << para << std::endl;
  
  my_print(paras...);
}
```

有一道算法题是，如果不用if-else for while求n个数的和，我们可以用这个的递归特性

```cpp
int g_sum = 0;
void my_print() { std::cout << "data:" << g_sum << std::endl; }
template <class T, class... args>
void my_print(T para, args... paras) {
  std::cout << para << std::endl;
  g_sum += para;
  my_print(paras...);
}

 my_print(1, 2, 3, 4, 5, 6, 7);
```

参考：  
[泛化之美–C++11可变模版参数的妙用](https://www.cnblogs.com/qicosmos/p/4325949.html)
