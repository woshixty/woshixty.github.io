转载自: [std::string的Copy-on-Write：不如想象中美好](https://www.cnblogs.com/promise6522/archive/2012/03/22/2412686.html)

# std::string的Copy-on-Write：不如想象中美好

Copy-on-write（以下简称COW）是一种很重要的优化手段。它的核心思想是懒惰处理多个实体的资源请求，在多个实体之间共享某些资源，直到有实体需要对资源进行修改时，才真正为该实体分配私有的资源。

COW技术的一个经典应用在于Linux内核在进程fork时对进程地址空间的处理。由于fork产生的子进程需要一份和父进程内容相同但完全独立的地址空间，一种做法是将父进程的地址空间完全复制一份，另一种做法是将父进程地址空间中的页面标记为”共享的“（引用计数+1），使子进程与父进程共享地址空间，但当有一方需要对内存中某个页面进行修改时，重新分配一个新的页面（拷贝原内容），并使修改进程的虚拟地址重定向到新的页面上。

**COW技术有哪些优点呢?**

1\. 一方面减少了分配（和复制）大量资源带来的瞬间延迟（注意仅仅是latency，但实际上该延迟被分摊到后续的操作中，其累积耗时很可能比一次统一处理的延迟要高，造成throughput下降是有可能的）

2\. 另一方面减少不必要的资源分配。（例如在fork的例子中，并不是所有的页面都需要复制，比如父进程的代码段(.code)和只读数据(.rodata)段，由于不允许修改，根本就无需复制。而如果fork后面紧跟exec的话，之前的地址空间都会废弃，花大力气的分配和复制只是徒劳无功。）

COW的思想在资源管理上被广泛使用，甚至连STL中的std::string的实现也要沾一下边。陈硕的这篇博客[《C++工程实践（10）：再探std::string》](http://www.cnblogs.com/Solstice/archive/2012/03/17/2403335.html)充分探讨了各个STL实现中对std::string的实现方式，其中g++ std::string和Apache stdcxx就使用了COW技术。（其他对std::string的实现包括eager copy和small string optimization，建议参考原博客，图文并茂十分清楚）

很简单一段代码，就能查看当前std::string实现是否使用了COW：

```plain
1 std::string a = "A medium-sized string to avoid SSO";
2 std::string b = a;
3 //a.data() == b.data()?
4 
5 b.append('A');
6 //a.data() == b.data()?
```

如果实现使用了COW，那么第一个比较会返回true，第二个比较会返回false。经测试libstdc++(gcc 4.5)确实使用了COW，而查看STL中string的源码，也确实采用了引用计数的手段。

但要注意，std::string的lazy-copy行为只发生在**两个string对象之间**的拷贝构造，赋值和assign()操作上，如果一个string由(const)char\*构造而来，则必然会分配内存和进行复制，因为string对象并不知道也无权控制char\*所指内存的生命周期。

```plain
1 std::string a = "Hello";
2 std::string b = "Hello";//Never COW!
3 assert(b.data() != a.data());
4 
5 std::string c = a.data();//Never COW!
6 assert(c.data() != a.data());
```

实际上，std::string c = a.data()确实是一种在字符串赋值时禁止COW行为的方法。

看起来使用COW管理string来减少不必要的拷贝似乎很有效，然而在多数C++ STL实现中，只有寥寥两种使用了COW，而同样著名的Visual C++(2010)和clang libc++却不约而同抛弃了COW，选择了SSO（small string optimization，足够小的字符串直接放在对象本身的栈内存中，避免了向Heap动态请求内存的开销）。

SSO对小字符串的高效是原因之一（程序中通常会有大量的短字符串），而COW本身的缺陷更是原因之一。

# **一、性能：for thread-safety!**

想要实现COW，必须要有引用计数（reference count）。string初始化时rc=1，每当该string赋值给了其他sring，rc++。当需要对string做修改时，如果rc>1，则重新申请空间并复制一份原字符串的拷贝，rc--。当rc减为0时，释放原内存。

基于”共享“和”引用“计数的COW在多线程环境下必然面临线程安全的问题。那么：

### std::string是线程安全的吗？

 在[stackoverflow](http://stackoverflow.com/questions/1594803/is-stdstring-thead-safe-with-gcc-4-3)上对这个问题的一个很好的回答：是又不是。

从在多线程环境下对共享的string对象进行并发操作的角度来看，std::string不是线程安全的，也不可能是线程安全的，像其他STL容器一样**。**

c++11之前的标准对STL容器和string的线程安全属性不做任何要求，甚至根本没有线程相关的内容。即使是引入了多线程编程模型的C++11，也不可能要求STL容器的线程安全：线程安全意味着同步，同步意味着性能损失，贸然地保证线程安全必然违背了C++的哲学：

> Don't pay for things you don't use.

但从不同线程中操作”独立“的string对象来看，std::string必须是线程安全的。咋一看这似乎不是要求，但COW的实现使两个逻辑上独立的string对象在物理上共享同一片内存，因此必须实现逻辑层面的隔离。C++0x草案(N2960)中就有这么一段：

> The C++0x draft (N2960) contains the section "data race avoidance" which basically says that library components may access shared data that is hidden from the user if and only if it activly avoids possible data races. 

**简单说来就是：你瞒着用户使用共享内存是可以的（比如用COW实现string），但你必须负责处理可能的竞态条件。**

而COW实现中避免竞态条件的关键在于：

1\. 只对引用计数进行原子增减

2\. 需要修改时，先分配和复制，后将引用计数-1（当引用计数为0时负责销毁）

## 先谈谈原子操作：

不同的体系结构一般会有不同的底层原语以支持原子操作。如Intel CPU本身就引入了#LOCK指令前缀，该前缀允许置于指定的操作（如算法指令、逻辑指令、bit指令、exchange指令等）之前使用，如lock inc会在执行inc指令时锁总线（锁定包含目标地址的一片内存区域，防止其他CPU在此期间的并发访问），从而序列化对同一地址的访问。

比起mutex之类的同步手段，原子操作自然要轻上不少，但比起普通的算术指令，依然算得上完全的重量级：

1\. 系统通常会lock住比目标地址更大的一片区域，影响逻辑上不相关的地址访问。

2\. lock指令具有”同步“语义，会阻止CPU本身的乱序执行优化。

Intel Developer's Manual vol 3的chapter 8 : Multiple-Processor Management中就有提到：

> "Locked instructions can be used to synchronize data written by one processor and read by another processor."

也就是会等待之前发出的load和store指令的完成（由于CPU store buffer的存在，如果数据之前没有依赖，不需要等待load和store的结果）

3\. 两个CPU对同一个地址进行原子操作，必然会导致cache-bounce。SMP系统中由于Cache一致性协议的存在，一个CPU对共享内存的修改必然会invalidate另一个CPU对该地址的cache，最终导致两个CPU对同一片内存不断”争夺“（cache不断被对方invalidate，需要重新从内存中读取），这是多线程编程中经典的[False Sharing](http://en.wikipedia.org/wiki/False_sharing)问题。

归根结底，COW为了保证”线程安全“而使用了原子操作，而原子操作本身的效率并不十分高。而且在多线程环境下，多个CPU对同一个地址的原子操作开销更大。**COW中”共享“的实现，反而影响了多线程环境下string”拷贝“的性能，并不scale。**

## 再谈谈操作顺序：

为了避免竞态条件，在需要修改string时，如果引用计数>1，必须先分配和拷贝，然后才能将引用计数减1。（而不能先减1再拷贝）

某些条件下，这样的操作序列会导致不必要的额外操作：

> string A在线程1中访问，string B在线程2中访问，string A 和 string B 共享同一片内容（rc = 2）
> 
> 假设当线程1操作string A时线程2恰好也在操作string B，双方发现该string的内容是共享的，都遵守先分配复制，后减引用计数的执行序列。（最终会有一方发现rc=0，销毁原string内容）。

到此为止，COW一共进行了**3次****内存分配和复制**（初始化时1次，修改时2次）和**1次内存释放**

实际上，如果没有使用COW技术，从string的初始化到目前为止也只进行了**2次内存分配和复制**（都是在初始化时进行）

# **二、”失效“问题：草木皆兵！**

假设当前的string实现是COW，考虑下面的代码：

```plain
std::string a = "some string";
std::string b = a;
assert(b.data() == a.data());// We assume std::string is COW-ed

std::cout << b[0] << std::endl;
assert(b.data() != a.data()); // Oops!
```

程序仅仅以”只读“的方式访问了b中一个字符，但b已经进行了一份私有的拷贝，a与b不再共享同一片内容。

由于string的**operator\[\]**和**at()**会返回某个字符的引用，而判断程序是否更改了该字符实在是超出string本身能力范围的东西，为了保证COW实现的正确性，string只得统统认定operator\[\]和at()具有修改的“语义”。

连**begin()/end()**也不能幸免，天晓得用户取得迭代器后会做什么！

这些无奈的”write alarm“实际上是由于**std::string本身接口的定义上没有对”只读“和”修改“语义做严格的区分。**

为此，Alexandrescu在它的”Scalable Use of STL“的演讲中对std::string的接口做了如下建议：

> 1\. offer set(n, c)
> 
> 2\. make default iterator non-mutating

仅仅是建议而已，实际上他本人反对使用COW：

> "The COW is dead, long live eager-copy"

#   

# **总结：**

1\. string的COW实现确有诸多的弊端，并不如想象中那般美好，也因此受到了Visual C++和clang++的抛弃，转而使用实现简单，且对小字符串更友好的SSO实现。

2\. Alexandrescue在他的“Scalable Use of STL”中建议对性能敏感的程序实现自己的string，比如针对string的长度进行选择优化（短字符串SSO，中等长度eager copy，长字符串COW），以及提供更加友好的接口等，并预期至少会有**10%**的整体性能提升。但我觉得，这实在不是普通程序员该干的事：暂且不论10%的底限能否达到，维护非标准的接口本身就是一笔重大的开销。

3\. 虽然我们的选择不多，但了解COW的缺陷依然可以使我们优化对string的使用：尽量避免在多个线程间false sharing同一个“物理string“，尽量避免在对string进行只读访问（如打印）时造成了不必要的内部拷贝。

最后发一句感慨：C++中的性能陷阱无处不在！

### 参考资料：

1. [《C++ 工程实践（10）：再探std::string》](http://www.cnblogs.com/Solstice/archive/2012/03/17/2403335.html) by 陈硕
2. [Is std::string thread-safe with gcc 4.3?](http://stackoverflow.com/questions/1594803/is-stdstring-thead-safe-with-gcc-4-3) (stackoverflow)
3. The scalable use of STL by Alexanderscue on 《C++ and beyond 2010》
4. [《Intel 64 and IA-32 Architectures Software Developer's Manual》 Volume 3](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
