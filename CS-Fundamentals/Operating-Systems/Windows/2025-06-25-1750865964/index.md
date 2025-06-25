转载自: [《windows核心编程系列 》 六 谈谈线程调度、优先级和关联性](https://www.cnblogs.com/weekbo/p/9054495.html)

# 《windows核心编程系列 》 六 谈谈线程调度、优先级和关联性

每个线程都有一个CONTEXT结构，保存在线程内核对象中。大约每隔20ms windows就会查看所有当前存在的线程内核对象。并在可调度的线程内核对象中选择一个，将其保存在CONTEXT结构的值载入cpu寄存器。这被称为上下文切换。大约又过20ms  windows将当前cpu寄存器存回内核对象，线程被挂起。Windows再次检查内核对象，并在可调度的内核对象中选择一个进行调度。此过程不断重复直到系统关闭。

Windows被称为抢占式多线程系统，系统可以在任何时刻停止一个线程而另行调度另外一个线程。我们对此可以有一些控制，但是权限很小。我们无法保证线程总在运行或者获得整个处理器。

由于windows并不是实时操作系统，我们无法保证在某一时间段一定在运行。

一般情况下，系统中的可调度线程很少。因为它们都在等待某个事件的到来。比如notepad程序在等待用户输入的时候它就是不可调度的。它在等待键盘的输入消息。当我们向notepad中输入时也并不意味着notepad会立即获得cpu时间。原因就是：windows并不是实时调度系统。

线程内核对象中有一个值表示挂起计数。调用CreateThread时系统创建线程内核对象，并把挂起计数初始化为1。这样cpu就不会调度它。在初始化之后，系统检查是否有CREATE_SUSPEND标识传入。如果有函数返回，进程仍保持挂起状态。否则将挂起计数递减为0，此时线程就可以被调度了。

通过创建一个挂起的进程或线程我们可以在它们执行任何代码前改变它们的环境，比如将其添加到作业中或是改变优先级。然后将它们设为可调度的。这可以通过调用ResumeThread函数。ResumeThread执行成功将返回前一次的挂起计数。失败则返回0xFFFFFFFF。

一个线程可以被挂起多次。除了在创建时传入CREATE_SUSPEND标识外，还可以调用SuspendThread函数。第一个参数为想要挂起的线程句柄。任何线程都可以挂起另一个线程。挂起n次的线程要想变为可调度的必须调用ResumeThread（）n次。

实际开发中，在调用SuspendThread时必须非常小心，因为我们无法知道线程此时在干什么。如果一个线程在分配堆中的内存，线程将锁定堆，其他想要分配堆的线程将被挂起。直到第一个线程分配完毕。如果第一个线程被挂起，将会出现死锁的情况。

由于进程不是cpu调度的单位，所以不存在挂起进程的概念。但是我们可以挂起进程中所有的线程。可以调用调试器函数WaitForDebugEvent函数。恢复时可以调用ContinueDebugEvent。

除了被别人调用SuspendThread挂起外，线程也可以告诉系统在一段时间内，可以将自己挂起，不需要调度。这可以调用Sleep实现。
```
void Sleep(DWORD dwMilliseconds);  
```

参数表示线程自己挂起的时间。但是实际的挂起时间只是近似于所设定的参数。Windows并不是实时操作系统，不能保证线程可以准时醒来。实际时间取决于系统中其他线程的运行情况。当为其传入0时，表示主调线程主动放弃本次时间片的剩余部分。注意是本次。

系统提供一个名为SwitchToThread函数，如果存在另一个可调度的线程，那么系统将让此线程运行。
```
BOOL SwitchToThread（）；
```

调用此函数时，系统查看是否存在急需cpu时间的饥饿线程。如没有则函数返回。如果存在，SwitchToThread将调度该线程。它与Sleep(0)很相似。区别在于SwitchToThread允许执行低优先级线程。

当我们需要计算线程执行某项任务的我的时间时，很多人习惯使用GetTickCount64函数。
```
ULONG start=GetTickCount64()；  
//do something.  
ULONG end=GetTickCount64();  
```

此段代码有个前提就是代码执行不会被中断。但是在抢占式OS中，线程可以随时被终止。执行上述代码的线程可能在执行第一个函数后就被挂起。一段时间后再次被调度。这时候时间就不准确了。

Windows提供了一个函数可以返回一个线程以获得cpu时间：   
```
BOOL GetThreadTime(  
    HANDLE hThread,  
    PFILETIME pftCreationTime,  
    PFILETIME pftExitTime,  
    PFILETIME pftKernelTime,  
    PFILETIME pftUserTime
);
```

第一个参数为想获得的线程句柄。

第二个参数返回（线程创建时间-1601年1月1日0:00）的秒数。单位是100ns。

第三个表示退出时间-1601年1月1日0:00的秒数。单位是100ns。

第四个表示线程执行内核模式下的时间的绝对值。单位是100ns。

第五个表示线程执行用户模式代码的时间的绝对值。单位是100ns。

类似的，GetProcessTime可以返回进程中所有线程的时间之和。

在进行高精度的计算时上述函数仍然不够。此时windows提供了以下函数：
```
BOOL QueryPerformanceFrequency(LARGE_INTEGER *pliFrequency)  
BOOL  QueryPerformanceCounter(LARGE_INTEGER *pliCount);  
```

这两个函数假设正在执行的线程不会被抢占。它们都是针对生命期很短的代码块。GetCPUFrequencyInMHZ可以获得cpu频率。

在windows定义的所有数据结构中，CONTEXT结构是唯一一个依赖于cpu的。我们可以通过调用GetThreadContext来获得当期cpu寄存器的状态。
```
BOOL GetThreadContext(
    HANDLE pThread,
    PCONTEXT pContext
);
```

第二个参数是CONTEXT结构指针。在分配CONTEXT结构后，需要初始化ContextFlag标志，表示以表示要获取哪些寄存器。函数执行后CONTEXT对象中就填入我们请求的成员。ContextFlag可以是：
```
CONTEXT_CONTROL表示控制寄存器。
CONTEXT_INTEGER表示整数寄存器。
CONTEXT_FLOAT 表示浮点寄存器。
CONTEXT_ALL 表示CONTEXT_CONTROL |CONTEXT_INTEGER|CONTEXT_SEGMENTS。
```

在调用GetThreadContext时，需要先调用SuspendThread。因为在调用GetThreadContext时系统可能正在执行那个线程，此时线程的上下文与获得的信息就不一致了。注意，它只能返回线程的用户模式上下文。如果当调用SuspendThread时线程正在内核模式运行，线程不会暂停，直到其返回用户空间。但是返回到用户控件后不会执行任何用户模式代码。

不仅仅能获得线程的进程上下文，我们还可以设置它。这可以调用：
```
BOOL SetThreadContext(
    HANDLE hThread,  
    CONST CONTEXT *pContext
);
```

GetThreadContext和SetThreadContext函数为我们提供了对线程许多控制的方法，但是需要小心使用。

线程优先级

前面提到的调度程序在调度另外一个线程之前，可以运行一个线程大约20ms的时间。但是这是所有优先级都相同的情况。实际上系统中的很多线程优先级是不同的，这将影响调度程序如何选择下一个要运行的线程。

Windows的线程优先级从0到31。每个线程都会分配一个优先级。当系统确定给哪个线程分配cpu时，它会首先查看优先级为31的线程，直至所有优先级为31的线程都被调度。然后再查看下一优先级线程。只要存在优先级为31的线程，系统就不会调度0-30级的线程。低优先级线程长时间得不到cpu时间，这被称为饥饿。这不经常出现，因为大多数线程都是不可调度的。

系统启动时会创建一个优先级为0的idle线程，整个系统只有它的优先级为0。它在系统中没有其他线程运行时将系统内存中所有闲置页面清0。

Windows中的线程优先级是由优先级类和相对线程优先级来确定的。系统通过线程的相对优先级加上线程所属进程的优先级来确定线程的优先级值。这个值被称为线程的基本优先级值。

Windows支持6个进程优先级类：idle ,below normal ,normal ,above normal,high和real-time。它们是相对与进程的。Normal最为常用，为99%的进程使用。

idle优先级类在系统什么都不做的时候运行的应用程序。如屏幕保护程序。real-time优先级类优先级别最高，但是没有开放给用户使用。因为此优先级类的程序会影响操作系统的任务。

Windows支持7个相对线程优先级：idle，lowest ，below normal，normal，above normal，highest和time-critical。这些优先级是相对于进程优先级的。大多数的线程使用normal优先级。

概括起来就是进程属于某个优先级类，另外还可以指定进程中线程的相对线程优先级。也就是说线程优先级是相对于进程优先级的。time-critical优先级对于real-time优先级类，优先级为31。相对于其他优先级类则为15。

需要注意的是进程优先级是抽象的概念，因为进程并不参与调度。

在优先级编程时，首先需要在调用CreateProcess时可以再fdwCreate参数中传入想要的优先级。fdwCreate可以是以下标识符：
```
real-time        REALTIME_PRIORITY_CLASS

high            HIGH_PRIORITY_CLASS

above normal     ABOVE_NORMAL_PRIORITY_CLASS

normal          NORMAL_PRIORITY_CLASS

below_normal    BELOW_NORMAL_PRIORITY_CLASS

idle             IDLE_PRIORITY_CLASS
```

进程运行后可以调用SetPrioritClass来改变进程优先级类。
```
BOOL SetPriorityClass(
    HANDLE hProcess,
    DWORD fdwPriority
);
```

可以调用GetPriorityClass来获得进程的优先级类。
```
DWORD GetPriorityClass(HANDLE hProcess);
```

上面是指定的进程优先级类，调用CreateThread创建线程时，它的线程优先级总是被设置为normal。可以调用以下函数来改变线程优先级：
```
BOOL SetThreadPriority(
    HANDLE hThread,
    int nPriority
);
```

nPriority可以是以下标识符：
```
time-critical       THREAD_PRIORITY_TIME_CRITICAL

highest           THREAD_PRIORITY_HIGHEST

above-normal      THREAD_PRIORITY_ABOVE_NORMAL

normal           THREAD_PRIORITY_NORMAL

below-normal      THREAD_PRIORITY_BELOW_NORMAL

lowest            THREAD_PRIORITY_LOWEST

idle              THREAD_PRIORITY_IDLE
```

但是在调用CreateThread时需要传入CREATE_SUSPEND，使线程暂停执行。

相应的可以调用int GetThreadPriority(HANDLE hThread);返回线程相对优先级。

Windows并没有返回线程优先级的函数，而是分别提供返回进程优先级类和相对线程优先级。

有些时候，系统也会提升一个线程的优先级。比如某个线程正在等待用户按键消息。当用户敲了一个键，系统会在线程的消息队列中放入一个WM_KEYDOWN消息。此时线程就变成可调度的了。键盘设备驱动程序将使系统临时提升线程的优先级。在该时间片结束后，系统会将线程的优先级值减一，第三个时间片执行时再减去一。直至保持基本优先级运行。

注意：线程的当前优先级不会低于进程的基本优先级。而且设备驱动程序可以决定动态提升的幅度。系统只提升优先级值在1~15的线程。这个范围被称为动态优先级范围。可以通过调用以下函数来禁止系统对线程优先级进行动态提升：
```
BOOL SetProcessPriorityBoost(
    HANDLE hProcess,
    BOOL bDisablePriorityBoost
);
```

此函数禁止动态提升此进程内的所有线程的优先级。
```
BOOL SetThreadPriorityBoost(
    HANDLE hThread,
    BOOL bDisablePriorityBoost
);
```

此函数禁止动态提升某个线程的优先级。

还有一种动态提升优先级的情况：检测到有饥饿情况出现时，也就是某个线程由于优先级低，而长时间无法得到调度时。系统就会动态提升此线程的优先级。系统允许它运行两个时间片。两个时间片结束之后立即恢复到基本优先级。

用户正在使用的窗口被称为前台窗口。这个进程就被称为前台进程。为了改进前台进程的响应性，windows会为前台进程中的线程微调调度算法。是前台进程的线程分配比一般情况下更多的时间片。

关联性

默认情况下，windows在分配cpu时采用软关联的方式。也就是说在其他因素相同的情况下，系统使线程在上一次运行的处理器上运行。这有助于重用仍在处理器高速缓存中的数据。

系统在启动时确定cpu数量。应用程序可以通过调用GetSysInfo来查询cpu的数量。如果需要限制一个进程的所有线程在某些cpu上运行，可以调用：
```
BOOL SetProcessAffinityMask(  
    HANDLE hProcess,  
    DWORD_PTR dwProcessAffinityMask
);  
```

第一个参数代表要设置的进程句柄。

第二参数是一个位掩码。代表线程可以在哪些cpu上运行。

注意子进程将继承父进程的关联性。

GetProcessAffinityMask返回进程的关联掩码。

相应的还可以设置某个线程只在一组cpu上运行：SetThreadAffinityMask。

有时候强制一个线程只在某个特定的cpu上运行并不是什么好主意。Windows允许一个线程运行在一个cpu上，但如果需要，它将被移动到一个空闲的cpu上。

要给线程设置一个理想的cpu，可以调用：
```
DWORD SetThreadIdealProcessro(
    HANDLE hThread
    DWORD dwIdealProcessor
);
```

dwIdealProcessor是一个0到31/63之间的整数。表示线程希望设置的cpu。可以传入MAXIMUM_PROCESSOR值，表示没有理想的cpu。