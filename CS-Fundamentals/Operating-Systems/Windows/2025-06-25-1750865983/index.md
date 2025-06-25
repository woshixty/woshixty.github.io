转载自: [《windows核心编程系列》 七 谈谈用户模式下的线程同步](https://www.cnblogs.com/weekbo/p/9054499.html)

# 《windows核心编程系列》 七 谈谈用户模式下的线程同步

用户模式下的线程同步

系统中的线程必须访问系统资源，如堆、串口、文件、窗口以及其他资源。如果一个线程独占了对某个资源的访问，其他线程就无法完成工作。我们也必须限制线程在任何时刻都能访问任何资源。比如在一个线程读内存时要限制其他线程对此块内存进行写入。

线程之间的通信很重要，尤其是在以下两种情况下：
- 1：需要让多个线程同时访问一个共享资源，同时不能破坏资源的完整性。
- 2：一个线程需要通知其他线程某项任务已经完成。

线程同步包括许多方面，windows提供了许多基础设施使线程同步变得容易。

用户模式下的线程同步：方法一，原子访问

线程同步的一大部分与原子访问有关。所谓原子访问，指的是一个线程在访问某个资源的同时能够保证没有其他线程会在同一时刻访问统一资源。

比如有全局变量a=0；有两个线程同时对一全局变量进行a++操作，然后返回结果。那么最后a是多少呢？让我们从汇编代码上分析一下。

A++编译器会编译成两行代码：
```
1.  Mov eax ,\[a\]  
2.  Inc eax  
3.  Mov \[a\],eax  
```

如果一切正常的情况下,两个线程顺序执行：
```
1.  Mov eax ,\[a\]  
2.  Inc eax  
3.  Mov \[a\],eax  
4.  Mov eax ,\[a\]  
5.  Inc eax  
6.  Mov \[a\],eax  
```

a的值是2，这是毫无疑义的。

但是由于windows是抢占式操作系统，系统可能在任何时刻暂停执行一个线程。基于此，前面的代码可能会按下面的顺序执行：
```
1.  Mov eax,\[a\]  
2.  Inc eax  
3.  Mov eax,\[a\]  
4.  Inc eax  
5.  Mov \[a\],eax  
6.  Mov \[a\],eax  
```

如果按照这种顺序执行，a的值最终应该是1，而不是2。这看起来很不可思议。但是由于我们无法对windows的调度进行控制，在这样的一个环境编程会让程序员崩溃。

为了解决这个问题，我们需要一种方法能够保证对一个值的递增操作是原子操作。Interlocked系列函数提供了我们需要的解决方案。所有这些函数会以原子方式来操控一个值。
```
1.  LONG InterlockedExchangeAdd(  
2.  PLONG volatile plAddend,  
3.  LONG lIncrement);  
4.  LONGLONG InterlockedExchangeAdd64(  
5.  PLONGLONG volatile pllAddend,  
6.  LONGLONG llIncrement);  
```

为上述两个函数传入一个长整形变量的地址和一个增量值，函数就会保证递增操作是以原子方式进行的。

前面的代码可以改成下面的代码：
```
1.  Long a;  
2.  DWORD WINAPI ThreadFunc2(PVOID pvParam)  
3.  {  
4.     InterlockedExchangeAdd(&a,1);  
5.     Return 0;  
6.  }  
```

改成上述代码后，对a的递增都是以原子方式进行。也就可以保证a的最终结果为2。如果只想以原子方式给一个值加1的话，可以使用InterlockedIncrement函数。

这些Interlock系列函数的工作机制取决于代码运行的cpu平台。如果是x86平台，那么Interlock函数会在总线上维持一个硬件信号，这个信号会阻止其他cpu访问同一个内存地址。

我们并不需要理解Interlock函数是如何工作的。无论编译器如何生成代码，无论机器上装了多少cpu，这些函数都能保证对值的修改是以原子方式进行的。

Interlock系列函数执行速度极快。通常只需要几个cpu周期，而且不需要在用户模式和内核模式进行切换。

使用这些函数也可以做减法，只要传入负值即可。

如果需要以原子方式执行变量交换可以调用：
```
1.  LONG InterlockedExchange(  
2.     PLONG volatile  plTarget,  
3.     LONG lValue);  
4.     LONGLONG InterlockedExchange64(  
5.     PLONGLONG volatile  plTarget,  
6.     LONGLONG lValue);  
7.     PVOID InterlockedPointer(  
8.     PVOID \*volatile ppvTarget，  
9.     PVOID pvValue);
```

InterloackedExchange和InterlockedExchangePointer会把第一个参数所指向的内存地址的当前值以原子方式替换为第二个参数指定的值

对于32位应用程序来说，它们都是替换的32位的值。而对于64位的应用程序InterlockedExchangePointer是替换的64位的值。因为此时指针也是64位的。

这两个函数都返回原来的值，在实现旋转锁时，InterlockedExchange函数非常有用。
```
1.  Bool use=false;  
2.  Void func()  
3.  {   
4.      While(InterlockedExchange(&use,true)==true)  
5.          Sleep(0);  
6.      //........  
7.      InterlockedExchange(&use,false);  
8.  }  
```

While循环不停的执行，把use的值设为true并检查原来的值是否为true。如果原来的值为false，则说明资源尚未被使用。于是将其设为使用中。如果原来的值是true则表明有其他线程正在使用该资源，调用sleep放弃该时间片的调度。

这里假定所有使用旋转锁的线程都是以相同的优先级运行的。但是对于单cpu的系统来说使用旋转锁是没有意义的。此处在检测到资源被占用时使用了sleep，可以睡眠一定随机的时间，这在一定程度上缓解了这种状况。

旋转锁假定被保护的资源只会被占用一段时间，与切换到内核模式然后等待相比，这种情况下以循环方式进行等待的效率会更高。许多开发人员会指定循环的次数，如果届时仍然无法访问资源，那么线程会切换到内核模式，并一直等待到资源可供使用为止。这就是关键段的使用方式。

在多处理器上使用旋转锁很有用，这是因为当一个线程在一个cpu上运行时，另一个线程可以再另一个cpu上循环等待。再次提醒，在单cpu的系统上循环锁毫无意义。

当cpu从内存中读取一个字节的时候，它并不是从内存中读取一个字节，而是取回一个高速缓存行。高速缓存行可能是32字节或是64字节。这取决于cpu。高速缓存行存在的目的是为了提高性能。这是根据程序的局部性原理。如果所有字节都在高速缓存行内那么cpu就不需要访问内存总线。

对于多处理器环境，高速缓存使得对内存的更新变得更加困难：
- 1：cpu1读取一个字节，这使得该字节以及与它相邻的字节被读取到cpu1的高速缓存中。
- 2：cpu2读取到同一字节，这使得该字节被读到cpu2的高速缓存中。
- 3：cpu1对内存中的这个字节进行修改，这使得该字节被写入到cpu1的告诉缓存中，并没有写回内存。
- 4：cpu2再次读取到同一字节。由于该字节已经在cpu2的高速缓存中，因此cpu2不需要在访问内存。但cpu2无法看到该字节在内存中的新值。

上述情况非常糟糕。但是cpu芯片的设计者早就考虑到了这种情况。当一个cpu修改了高速缓存行的一个字节时，机器中的其他cpu会收到通知，并将自己的高速缓存行作废。因此在刚才的情形中，cpu2的高速缓存就作废了。Cpu1必须将它的高速缓存写回内存，cpu2必须重新访问内存来填满它的高速缓存行。我们可以看到虽然高速缓存可以提高性能，但是在多处理器机器上它同样能够损伤性能。

为了提高性能，我们应该根据高速缓存的大小来将应用程序的数据组织在一起，将数据与缓存行的边界对齐。并把只读数据与可读写数据分别存放。

使用GetLogicalProcessorInformaiton函数可以获得cpu高速缓存行的大小。我们可以使用_declspec(align(#))指示符来对字段对齐加以控制。说了那么多的措施，其实最好的方法就是始终让一个cpu访问数据或只让一个线程访问数据就可以完全避免高速缓存行的问题。

如果我们只需要以原子方式修改一个值，那么Interlock系列函数非常好用。但是大多数情况下我们需要处理的数据结构往往要比一个简单的32位值或64位值复杂的多。为了能够以原子的方式来访问复杂的数据结构，我们可以使用windows使用的其他特性。

使用旋转锁不停的访问是非常糟糕的一种方式，最好的情况下就是当线程想要访问共享资源时，线程必须调用操作系统的一个函数，告诉操作系统自己等待什么资源。如果此时资源不可用，此线程将会进入等待状态，不可被调度。如果操作系统检测到资源已经可以使用了，系统就会将此线程设为可调度状态并访问此资源。

volatile关键字告诉编译器这个变量可能会被应用程序之外的其他东西修改，不要对该变量做任何形式的优化，而是始终从内存中读取此值。如果不对一个变量加上volatile限定符，编译器可能会对C++代码进行优化，如它会将变量值载入到寄存器中，以后的操作都是对寄存器中的值进行操作，并不会多次访问内存。对一个结构加上volatile限定符等于给结构中的所有成员都加入volatile限定符。

关键段（Critical Section）

关键段是一小段代码，它在执行之前需要独占对一些共享资源的访问权。这种方式可以让多行代码以原子方式访问，在当前线程离开关键段之前，系统不会调度任何其他线程访问该关键段。

比如在一个链表管理的例子中，如果对链表的访问没有同步，那么一个线程可能会在另一个线程在链表中查询时向链表添加元素。如果两个线程同时向链表中添加元素情况会更糟。而使用关键段可以有效地防止以上各种情况。

要使用关键段首先需要定义CRITICAL_SECTION结构。然后把任何需要共享的代码放在EnterCriticalSection和LeaveCriticalSection之间。如:
```
1.  Int g_a=0;  
2.  CRITICAL_SECTION cs;  
3.  DWORD WINAPI ThreadProc1(PVOID)  
4.  {  
5.      EnterCriticalSection(&g_a);  
6.      for(int i=0;i<100;i++)  
7.          g_a++;  
8.      LeaveCriticalSection(&cs);  
9.      return 0;  
10.  }  
11.  DWORD WINAPI ThreadProc2(PVOID)  
12.  {  
13.     EnterCriticalSection(&g_a);  
14.     for(int i=0;i<100;i++)  
15.         g_a++;  
16.     LeaveCriticalSection(&cs);  
17.     return 0;  
18.  }  
```

作者使用了一个非常形象的比喻。CRITICAL_SECTION结构就像一个卫生间，卫生间内的区域就是要保护的资源。在同一时刻只允许一个人进入卫生间内。如果有多个区域需要保护可以分别定义多个CRITICAL_SECTION结构。调用EnterCriticalSection传入CRITICAL_SECTION结构的地址，这个结构标识要访问的保护资源，也就相当于卫生间。当一个人想上卫生间时，必须首先检查卫生间门上的占用标识,看是否有人占用。如果此时无人，那么EnterCriticalSection将允许调用线程进入卫生间。如果卫生间已有人占用，那么调用线程必须在门外等待。如果一个人使用过卫生间后，他必须把卫生间改为未占用状态。LeaveCriticalSection告诉系统它已经离开了所占用的资源。如果忘记调用LeaveCriticalSection，系统会一直让等待进入此卫生间的人在门外等待。

关键段由于在内部使用了Interlock系列函数因此执行速度非常快。它的缺点是不能在多进程之间对线程进行同步。而信号量和事件则可以。

一般情况下CRITICAL_SECTION结构会作为全局变量来分配，这样进程内的所有线程都可以通过该变量来访问这些结构。实际使用中将此结构作为局部变量、从堆中分配或者是类的私有成员也都是可以的。

但有两个必要条件：
- 第 一：想要访问资源的线程必须知道用来访问资源的CRITICAL_SECTION结构的地址。
- 第二：在任何线程访问被保护的资源之前，必须对CRITICAL_SECTION结构进行初始化。初始化调用：
```
VOID InitializeCriticalSection(PCRITICAL_SECTION pcs);  
```
此函数会设置CRITICAL_SECTION结构的一些成员。如果这些成员没有经过初始化，结果将是不可预料的。

当线程不需要访问共享资源时，应该调用以下函数来清理CRITICAL_SECTION结构：
```
VOID DeleteCriticalSection(PCRITICAL_SECTION pcs);
```

在访问共享资源调用EnterCriticalSection，该函数将执行下面的动作：
- 1：如果没有线程在访问资源，该函数会更新成员变量，表示线程已经获得对资源的访问权，同时更新调用线程被准许访问的次数。线程可以继续执行。
- 2：如果有其他线程占用资源，该函数会使用一个事件内核对象把调用线程切换到等待状态。当占用线程调用LeaveCriticalSection时，系统会自动更新CRITICAL_SECTION的成员变量并将挂起的线程切换到可调度状态。

为了防止当资源被其他线程占用时，主调线程被挂起，可以调用TryEnterCriticalSection函数，该函数将会检测此时主调线程是否可以访问共享资源。如果资源被占用该函数返回false。否则返回true。

如果检测到资源此时已被占用，主调线程这是可以做其他事情，而不是被挂起。由于TryEnterCriticalSection函数会更新CRITICAL_SECTION结构的某些成员，因此需要对应一个LeaveCriticalSection函数。

读写锁（RWLock）

读写锁也可以对一个资源进行保护。但是它跟关键段有所不同，读写锁允许我们区分那些想要读取资源的线程和更改资源的线程。让所有的读取资源的线程同时工作是可行的，因为读取不会破坏数据。只有当写入时才需要对写入线程进行同步。写入线程必须独占资源，它工作时无论是读取还是写入线程都必须等待。

使用读写锁之前，需要分配一个SRWLOCK结构，并调用InitializeSRWLock初始化它。
```
Void InitializeSRWLock(PSRWLOCK SRWLock);  
```

初始化完成之后，写入者线程可以调用AcquireSRWLockExclusive，将SRWLock对象地址传入，获得对被保护资源的独占访问。
```
Void AcquireSRWLockExclusive(PSRWLOCK SRWLock);  
```

独占访问结束后，需要调用：
```
Void ReleaseSRWLockExclusive(PSRWLOCK SRWLock);  
```

解除对资源的独占。

对于读取者线程，可以通过调用：
```
Void AcquireSRWLockShared(PSRWLOCK SRWLock);  
```

获得对资源的共享访问。
```
Void ReleaseSRWLockShared(PSRWLOCK SRWLock);  
```

释放对资源的共享。

读写锁不能为了多次写入资源而多次锁定资源，然后再多次调用ReleaseSRWLock\*来释放对资源的锁定 。

多线程会导致一些同步开销。如果使用单线程访问一个资源需要x ms，那么使用两个线程花费的时间将会>2x。因为除了同步开销外，还有多个cpu之间必须相互通信以维护高速缓存的一致性。Cpu个数越多，这种花费就越多。

读写锁执行读取操作的性能要优于写入操作。这是因为两个线程可以同时读取，但是需要写入的时候只能一个一个写入。

对比各种线程同步方法，读写锁和关键段性能差不多，有些情况下读写锁甚至更优，而且还允许多个线程同时读取。因此推荐使用读写锁。使用互斥量性能是最差的。这是因为等待互斥量以及后来的释放互斥量需要线程调用API，而这又会导致在用户模式和内核模式之间切换。在下一章介绍的使用内核对象进行同步时还会详细介绍。

使用读写锁也可以解决生产者-消费者问题。

如果读取者线程没有数据可读，它应该将锁释放并等待，直到生产者写入新数据为止。如果写入者将缓冲区写满，它同样应该释放锁并进入睡眠状态。要实现这样的线程同步是很复杂的，windows为我们提供一些函数简化了这些操作。

Windows提供SleepConditionVariableCS或SleepConditionVariableSRW函数，等待条件变量。线程在等待该条件变量时，会以原子方式把锁释放并将自己阻塞，直到该条件变量被触发时为止。
```
1.  Bool SleepConditionVariableCS(  
2.    PCONDITION_VARIABLE pConditionVariable,  
3.    PCRITICAL_SECTION pCriticalSection,  
4.    DWORD dwMilliseconds);  
5.  Bool SleepConditionVariableSRW(  
6.    PCONDITION_VARIABLE pConditionVariable,  
7.    PSRWLOCK pSRWLock,  
8.    DWORD dwMilliseconds  
9.    ULONG Flags);
```

pConditonVariable指向一个以初始化的条件变量，调用线程将等待该条件变量。第二个参数指向一个关键段或是SRWLock对象。该关键段或SRWLock用来同步对共享资源的访问。Flags指定一旦条件变量被触发，线程将以何种方式获得锁。对读取者线程来说应该传入CONDITION_VARIABLE_LOCKMODE_SHARED表示希望共享对资源的访问。对于写入者线程应该传入0，表示独占资源。

dwMilliseconds表示我们希望线程花多少时间来等待条件被触发。在指定的时间用完时，如果条件变量尚未被触发，函数返回false，否则为true。

当另一个线程检测到相应的条件已经满足时，比如存在一个元素可以让读取者线程读取。它会调用WakeConditionVariable或WakeAllConditionVariable，触发条件变量。这样调用Sleep\*函数而阻塞在该条件变量的线程就会被唤醒。
```
1.  Void WakeConditonVariable(  
2.     PCONDITION_VARIABLE ConditionVariable);  
3.  Void WakeAllConditionVariable(  
4.     PCONDITION_VARIABLE ConditionVariable);
```

WakeConditionVariable会使SleepConditionVariable\*等待的同一个条件变量被触发的线程得到锁并返回。当此线程释放这个锁的时候，不会唤醒其他正在等待此条件变量的线程。