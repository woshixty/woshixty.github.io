转载自: [《windows核心编程系列》五 谈谈线程基础](https://www.cnblogs.com/weekbo/p/9054489.html)

# 《windows核心编程系列》五 谈谈线程基础

与前面介绍的进程一样，线程也有两部分组成。一个是线程内核对象。它是一个数据结构，操作系统用它来管理线程以及用它来存储线程的一些统计信息。另一个是线程栈，用于维护线程执行时所需的所有函数参数和局部变量。位于同一个进程的线程共享进程的地址空间且它们共享进程句柄表。因为句柄表是针对进程的。进程需要很多的系统资源，而线程仅仅需要一个线程内核对象和线程栈就可以了，因此线程比进程的开销要小得多。采用多线程来处理问题也是理所当然的了。

采用多线程可以提高程序的执行效率，但是多线程也存在很多问题。在尝试使用多线程时如果处理不当还可能会引入新的问题。如同步问题。

每个线程都需要一个入口点函数。这是线程执行的起点。主线程的入口点函数是_tmain或_tWinmain。如果在进程中创建新线程必须提供自己的入口点函数。

形如：
```
DWORD ThreadProc(PVOID pvParam)
{
    int temp=(int)pvParam;
    ....
    return 0;
}
```

线程函数可以是任何我们希望它执行的任务，最终线程函数会终止并返回。类似于进程内核对象，如果线程内核对象使用计数变为0，则会被销毁。默认情况下主线程的入口点函数必须命名为main,wmain,WinMain或wWinMain。我们可以通过设置/ENTRY:链接器选项来指定另一个函数作为入口点函数。

线程函数只有一个参数，其意义可由我们定义。可以为其传递一个值，也可以将其作为某个数据结构的指针。这需要在线程函数内部做类型转换。线程函数必须返回一个值，它的值传递给ExitThread，作为线程的退出代码。

线程函数尽可能地使用局部变量或函数参数，它们是在线程栈上创建的。不太可能被其他线程破坏。使用静态变量或全局变量时其他线程可以访问这些变量，这会导致同步和互斥问题。

接下来介绍线程的创建，与其他内核对象一样，它也有对应的Create\*函数。
```
HANDLE CreateThread(
    PSECURITY_ATTRIBUTE psa,
    DWORD chStackSize,
    PTHREAD_START_ROUTINE pfnStartAddr,
    PVOID pvParam,
    DWORD dwCreateFlag,
    PDWORD pdwThreadID
);
```

调用此函数时，系统会创建线程内核对象，这个线程内核对象不是线程本身而是一个小的数据结构，操作系统用这个结构来管理线程。系统从进程的地址空间中分配内存给线程栈使用，新线程与负责创建的线程在相同的进程上下文运行。因此，新线程可以访问进程内核对象的所有句柄、进程中的所有内存以及同一个进程中其他所有线程的栈（虽然线程栈属于每个线程独有，但是所有线程都在同一个地址空间，也可以被其他线程访问）。这样一来多个线程很容易的相互通信。

psa参数指明内核对象的安全属性。传入NULL，表示使用默认安全属性。如果希望子进程能继承到这个对象的句柄，必须指定一个SECURITY_ATTRIBUTES结构，并将该结构的bInherithandle成员初始化为true。表明此内核对象是可继承内核对象。

cbStackSize参数指定线程栈的大小。每个线程都拥有自己的栈。对于主线程cbStackSize使用的是保存在PE文件头的内部的值。可以使用链接器/STACK开关控制这个值，如：`/STACK:\[reserve\] \[,commit\]`

reserve参数用于设置为线程栈预定多少空间。默认为1MB。commit参数指定最初应为栈预留的地址空间调拨多少物理内存。默认是一个页面。随着线程的执行需要的空间可能不止1MB，如果栈溢出就会产生异常。系统捕获异常并未已预订的空间调拨另一个页面。关于线程栈的内容请参考后面的内容。

当我们在CreateThread中为cbStackSize传入0时，系统会为线程栈预定空间并调拨所有的物理内存。这里只是说会为所有的已预定空间调拨内存，至于预定空间是多大，要么由/STACK链接器开关指定，要么由cStackSize指定。取其中较大的一个。预订的地址空间设定了栈空间的上限，也就是说在栈上限内如果物理内存不够使用还可以调拨，但是一旦达到这个上限就不会再为其分配内存。这可以防止无穷递归。

pfnStartAddr参数指定线程函数的地址，pvParam是传给线程函数的参数。创建多个线程时可以让他们使用同一个函数地址。前面提过使用多线程可能会导致很严重的同步问题，可以采取第八章和第九章的线程同步技术解决。

dwCreateFlags参数控制线程的创建。如果其为0，线程创建之后立即就可以调度。如果值为CREATE_SUSPEND系统将创建并初始化线程，但是会暂停线程的执行，直到调用ResumeThread恢复期运行。

pdwThreadID参数存储线程的ID，它是一个DWORD类型的指针。如果传入NULL，表明我们对线程ID不感兴趣。

线程可以通过以下四种方式终止运行：

1：线程函数返回。

2：线程调用ExitThread杀死自己。

3：其他线程调用TerminateThread。

4：进程终止导致线程终止。

第一种方法是最保险的方式，它可以保证线程的所有资源都会被正确清理。如线程创建的所有C++对象的析构函数都会被调用、线程栈内存被释放、线程的退出代码设为线程函数的返回值。线程内核对象的使用计数被递减。

ExitThread也会导致线程终止运行，同时系统也会清理该线程使用的所有操作系统资源，但是与自然退出线程函数不同，所有线程创建的C++对象的析构函数不会被调用。至于为什么不会被调用待会儿会专门介绍。线程函数正常退出时（对象被析构）会退出到其他函数，在此函数内ExitThread会被调用，此时这是正常的。

TerminateThread可以杀死一个线程，它的第一个参数标识要杀死的线程的句柄，第二个参数为线程退出代码。这个函数不推荐使用。因为它很不负责。被终止的线程在得不到任何通知情况下线程栈不会被销毁。也就相当于把人杀了就把尸体扔那，占用地方。线程栈不会被销毁，还在占用内存，这当然就属于内存泄露。但是这些内存还不能被其他线程访问，如果访问的话会导致访问违规。虽然TerminateThread不会释放线程栈，但是进程结束时栈空间会被清理。   

线程终止运行时，线程所拥有的所有用户对象句柄会被释放。其退出代码从STILL_ACTIVE变为传给ExitThread或Terminatethread的代码。线程内核对象变为触发状态且引用计数减一。如果该线程是进程的最后一个活动线程，也会导致进程终止。
```
BOOL GetExitCodeThread(
    HANDLE hThread,
    PDWORD pdwExitCode
);
```

线程终止后，其内核对象不一定被释放。其他线程可以调用GetExitCodeThread来检查hThread所标示的线程是否已终止运行。如果pdwExitCode返回STILL_ACTIVE表明线程尚未终止。否则返回其退出代码。也可以使用WaitForSingleObject来判断线程内核对象句柄是否是已触发状态。

CreateThread函数创建线程内核对象，该对象的最初使用计数为2。一个是该函数返回的句柄对该内核对象的引用，一个是线程本身也占有一个引用。暂停计数被置为1，退出代码被设为STILL_ACTIVE，对象处于未触发状态。线程上下文反映了当前线程上一次的执行时的cpu寄存器的状态，它保存在CONTEXT结构中，该结构保存在线程内核对象中。

线程内核对象被初始化时，CONTEXT结构中堆栈指针SP被设为pfnStartAddr，而指令指针寄存器IP被设为RtlUserThreadStart函数。该函数由NTDLL.dll导出。它的参数是真正的线程函数地址和线程参数。

因为新线程的指令指针寄存器IP被设为RtlUserThreadStart，因此这个函数实际就是线程开始执行的地方。这个函数接受两个参数，也许你会以为这个函数是从另一个函数调用的。但是事实并非如此，新线程之所以在此处开始执行并访问这两个参数，是因为在线程初始化时，CONTEXT结构的IP被设为RtlUserThreadStart。此函数并没有在其他函数中调用也就没有返回地址。这一点要明白。
```
VOID RtlUserThreadStart（PTHREAD_START_ROUTINE pfnStartAddr,PVOID pvParam)
{
    try {
        ExitThread((pfnStartAddr)(pvParam));
    }
    exception(UnHandledExceptionFilter(GetExceptionInformation())
    {
        ExitProcess(GetExceptionCode());
    }
}
```

在执行RtlUserThreadStart时，它会设置结构化异常处理，这样一来线程执行期间所产生的任何异常都能得到系统的默认处理。CreateThread在函数中调用，并传递给pvParam参数。在线程函数返回时（局部的C/C++对象析构）RtlUserThreadStart函数调用ExitThread(在线程函数之外调用的，C/C++对象可以被析构)，将线程函数的返回值传给它，此时线程内核对象的使用计数被递减，然后线程终止。在RtlUserThreadStart函数中线程要么执行ExitThread要么执行ExitProcess，这就是说RtluserThreadStart不能正常的返回。本身就没有其他函数调用它，也就没有返回地址。正常返回也会出错。

进程的主线程在初始化时它的IP也会初始化为RtlUserThreadStart函数。该函数执行时会调用C/C++运行库的启动代码，启动代码调用入口函数。注意：线程永远不会返回到RtlUserThreadStart函数中。

以前，单线程和多线程的库是分开的。之所以采取这个设计是因为标准C运行库是在1970年之后左右发明的，比多线程要早得多。那时标准C根本没有考虑到多线程的情况。采用多线程后，单线程的库这可能导致很多问题。

如C运行库的全局变量errno，有的函数在出错时会设置该变量。如果在一个线程的函数设置过errno后被挂起，然后其他线程开始执行，新线程也有函数设置errno。那么当第一个线程返回时errno，已经不是第一个线程中的函数设置的值了。这时的errno就不在正确了。为此，必须有某种机制能够让一个线程引用它自己的errno。这仅仅证明了标准C/C++库最初不是为多线程应用程序而设计的。

为保证C和C++应用程序正常运行，必须创建一个数据结构，并使之与使用C/C++运行库函数的每个线程关联。在调用C/C++运行库函数时，那些函数必须知道去查找主调线程的数据块，修改或访问它们，从而避免影响其他线程。

系统在创建新线程时，并不知道应用程序是使用C/C++来写的，也不知道你调用的函数并非天生就是线程安全的。因此在创建线程时一定不要调用操作系统的CreateProcess函数。必须调用C/C++运行库函数_beginthreadex。

_beginThreadex函数的参数列表跟CreateThread一样，但是参数名称和类型并不完全一样。这是因为Microsoft的C/C++运行库开发组认为C/C++运行库不应该对Windows数据类型有任何的依赖。

在_beginthreadex内部，申请了_tiddata内存块，它是在C/C++运行库的堆中分配的。传给_beginthreadex的线程函数的地址保存在_tiddata内存块中。在内部CreateThread会被调用，但是此时传给它的地址是_threadstartex而非pfnStartAddr。参数的地址是_tiddata结构的地址，而非pvParam。虽然传给CreateThread的线程函数是_threadstartex，但是跟前面介绍的一样，实际上仍然是RtlUserThreadStart先执行，然后再执行_threadstartex。所有的CreateThread都是这样处理。这一点要注意。

在_threadstartex函数中，参数是_tiddata结构的地址，TlsSetValue是系统API。它用来将_tiddata块与新建线程关联起来。然后_callthreadstartex函数被调用。此函数处理结构化异常。由于真正的线程函数保存在_tiddata内存块中，在此函数内又会调用真正的线程函数。线程函数返回后_endthreadex被调用，该函数获取主调线程_tiddata数据块的地址，然后释放。释放后调用操作系统提供的ExitThread销毁线程。

整个过程弄清楚了，接下来总结一下：什么情况C/C++对象的析构函数不会调用，什么情况下_tiddata数据块不会被释放。函数执行时，局部的C/C++对象的构造函数被调用，函数返回时析构函数被调用。由于ExitThread和TerminateThread不仅仅会导致进程终止还会阻止函数返回。也就说线程函数执行到ExitThread或TerminateThread时就结束了。由于没有函数的返回局部的C/C++对象的析构函数当然没有被调用。在多线程开发时，_beginthreadex为每个线程准备了一个_tiddata数据块，用于存储局部于线程的数据，防止多线程互相影响，这个数据块是从堆中分配的。对应的应该调用_endthreadex，否则将会导致内存泄露。切不可与CreateThread或ExitThread、TerminateThread混用。

在创建线程时应该使用_beginthreadex，经常见到CreateThread大部分情况下这是可以的。当使用CreateThread时，一个线程调用一个需要_tiddata结构的C/C++运行库函数时会发生下面的情况。首先C/C++运行库尝试取得线程数据块的地址，如果NULL被当做_tiddata数据块的地址返回，表明主调线程没有与之关联的_tiddata数据块。此时，C/C++运行库函数会为主调线程分配并初始化一个_tiddata块，然后将此块与线程关联。而且只要线程还在这个块就一直与线程进行关联。此后C/C++运行库可以使用_tiddata数据块，此后的任何C/C++运行库函数也可以使用。也就是说即使你使用CreateThread，C/C++运行库检测到NULL时，也会分配并初始化一个_tiddata块，只是在调用ExitThread时此内存块不会被调用，导致内存泄露。另一个问题是结构化异常没有就绪，当使用C/C++运行库的signal函数时将会导致进程终止。

一般建议使用_beginthreadex和_endthreadex配合使用。

通过调用GetCurrentProcess和GetCurrentThread可以获得主调进程和线程的句柄。但是它们返回的仅仅是一个伪句柄。。GetCurrentProcess返回的当前进程句柄值为-1，而GetCurrentThread返回的值为-2。它们不在句柄表中。伪句柄不能用于跨进程共享，即使传递过去也是调用其他进程或线程句柄。所以在将当前句柄进行跨进程共享时，要先经过转换。这可以通过DuplicateHandle将伪句柄转换为真正的句柄。如：
```
DuplicateHandle(
    GetCurrentProcess(),
    GetCurrentThread(),
    GetCurrentProcess(),
    &hThreadParent
);
```
  
注意该函数会导致线程句柄的使用计数加一。不使用时应调用CloseHandle关闭对该句柄的引用。
