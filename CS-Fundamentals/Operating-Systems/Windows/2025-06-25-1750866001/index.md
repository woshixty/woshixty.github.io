转载自: [《Windows核心编程系列》 八 谈谈用内核对象进行线程同步](https://www.cnblogs.com/weekbo/p/9054503.html)

# 《Windows核心编程系列》 八 谈谈用内核对象进行线程同步

前面我们介绍了用户模式下线程同步的几种方式。在用户模式下进行线程同步的最大好处就是速度非常快。因此当需要使用线程同步时用户模式下的线程同步是首选。

但是用户模式下的线程同步也存在缺点。如InterLocked系列函数只能对一个值进行操作。关键段虽然可以对一段代码进行操作，但是只能对同一个进程内的线程进行同步。为了解决上述问题，我们将会讨论使用内核对象进行线程同步。

与用户模式下的速度快相比较，使用内核对象进行线程同步性能很低。原因就是：在创建或清除内核对象时调用线程必须从用户态切换到内核模式。这种切换非常耗时。除此之外还会造成刷新高速缓存以及未命中。这将耗费更多的cpu时间。

前面我们已经介绍了几种内核对象，包括：进程、线程以及作业对象。除此之外可以进行线程同步的内核对象还包括：事件、计时器、信号量和互斥量等。

几乎所有的内核对象都可以进行线程同步。这些内核对象都包括两种状态：一种是触发状态，另一种是未触发状态。例如，进程和线程对象在刚创建的时候是未触发状态，当进程和线程终止时就变成了触发状态。此状态转换是不可逆的。

之所以能进行从未触发到触发的转变是因为内核对象内部有一个布尔变量。系统创建内核对象时会把它的初始值设为false，表示未触发 。内核对象状态的切换就是对此值进行切换。

所以当我们判断一个内核对象是否处于触发状态只需要检测此变量的值就可以了。

WaitForSingleObject系列等待函数。

该函数可以使一个线程进入等待状态，直到它所等待的内核对象被触发为止。

```
DWORD WatiForSingleObject(
    HANDLE hObject,
    DWORD dwMilliseconds
);
```

hObject标识要等待的内核对象。这个对象可以处于触发或未触发状态。如果对象处于未触发状态，主调线程将会一直等待，直到它编程触发状态时为止。如果调用函数时，内核对象已经被触发，那么等待函数直接返回。

第二个参数dwMilliseconds指定主调线程最多愿意花多长时间来等待对象被触发。当指定INFINITE时则告诉系统希望线程永远等待。也可以传入其他值。以微秒为单位。如果在指定的时间内内核对象被触发，则等待函数返回WAIT_OBJECT_0。否则将返回WAIT_TIMEOUT。

例子：
```
DWORD dw=WaitForSingleObject(hProcess,5000);  
switch(dw)  
{  
    cse WAIT_OBJECT_0:  //规定的时间内等待成功。
        break;
    case WAIT_TIMEOUT:  //超时。
        break;
    case WAIT_FAILED:   //指定的句柄无效。
        break;
    Default:
        break;
}
```

可以调用GetLastError获取更详细的出错信息。 

WaitForMultipleOBjecs函数与WaitForSingleObject相类似。唯一的不同是他允许线程同时等待多个内核对象。
```
DWORD WaitForMultipleOBjecs(  
    DWORD dwCount,
    CONST HANDLE*pbObject,
    BOOL bWaitAll,
    DWORD dwMilliseconds
);
```

dwCount指定等待内核对象的数量。

pbObject指向一个存储所有需要等待内核对象的数组。

bWaitAll指定是否等待所有内核对象都触发。当其为true时，只有当所有内核对象都变成触发态时函数返回。否则，只要有一个内核对象变成触发态等待函数就返回。

dwMillisecond是指定等待时间与WaitForSingleObject相同。

bWaitAll为true和false时函数返回值意义不同。

当为true时，函数返回时所有对象都被触发。这与WaitForSingleObject的返回值意义相同。

当为false时，返回值将会标识那个对象被触发从而导致等待函数返回。

例子：
```
HANDLE h[3];  
H[0]=hProcess1;  
H[1]=hProcess2;  
H[2]=hProcess3;  
DWORD dw=WaitForMultipleOBjecs(3,H,false,5000);  
switch(dw)
{
    case WAIT_OBJEC_0://第一个对象被触发。
        break;
    case WAIT_OBJEC_0+1://第二个对象被触发。
        break;
    case WAIT_OBJEC_0+2://第三个对象被触发。
        break;
    case WAIT_TIMEOUT://超时
        break;
    case WAIT_FAILED://句柄无效。
        break;
}
```


事件内核对象

事件内核对象是在线程同步时比较常用的内核对象。

一个事件内核对象的触发表示一个操作已经完成。有两种类型的事件内核对象：手动重置事件和自动重置事件。

所谓自动重置事件，就是当线程等待自动重置事件成功后，对象将自动重置成非触发状态。这也正是自动重置事件的由来。

而手动重置事件必须手动设置成自动重置事件。

一个手动重置事件对象触发时，所有等待此对象的线程都变成了可调度状态。而一个自动重置事件变成触发时在所有等待该内核对象触发的线程中只有一个可以变成可调度状态。

使用CreateEvent可以创建一个事件内核对象。
```
HANDLE CreateEvent(
    PSECURITY_ATTRIBUTES psa,  
    BOOL bManualReset,  
    BOOL bInitialState,  
    PCTSTR pszName
);
```
  
psa表示内核对象的安全属性。赋值NULL，表示使用默认的安全属性。(前面对安全属性有详细的介绍)。

bManualReset指定要创建的内核对象是手动重置还是自动重置内核对象。

bInitialState指定事件对象的初始状态。触发或未触发。

pszName指定该内核对象的名称。可用于跨进称共享内核对象。

当调用此函数成功时，CreateEvent将返回一个事件内核对象的句柄。

此外还有CreateEventEx函数对对象的权限进行控制，此处不再介绍。

前面我们提到过内核对象可以实现跨进称线程同步。要实现此功能，必须能在其他进程中访问此内核对象。这可以通过一下几种方式：
- 1：调用CreateEvent时，指定pszName与已存在的事件内核对象同名。此时与OpenEvent功能相同。
- 2:使用继承。
- 3：使用DuplicateHandle函数。
```
HANDLE OpenEvent(
   DWORD dwDesiredAccess,  
   BOOL bInherit,  
   PCTSTR pszName);
```

该函数会打开pszName指定名称的已存在的内核对象。如果为CreateEvent指定pszName，此名称已存在且类型相同都为事件内核对象，则CreateEvent也仅仅是打开此内核对象获得句柄。只有当此名称的内核对象不存在或类型不同时才创建新的内核对象。

不使用内核对象时要调用CloseHandle关闭对它的引用。

SetEvent可以将事件内核对象变为触发态。
```
BOOL SetEvent(HANDLE hEvent)；  
```

  
ResetEvent可以把事件内核对象变为未触发状态。
```
BOOL ResetEvent(HANDLE hEvent);  
```


事件内核对象在线程同步中经常使用。接下来介绍一个例子：
```
HANDLE g_hEvent;  
int main()  
{  
    g_hEvent=CreateEvent(NULL,true,false,NULL);  
    HANDLE hThread=CreateThread(NULL,0,Thread1,NULL,0,NULL);  
    //打开文件并读取内存。  
    SetEvent(g_hEvent); //通知Thread1开始运行。  
    //其他操纵。  
}

DWORD WINAPI Thread1(PVOID param)  
{
    WatiForSingleObject(g_hEvent,INFINITE);
    //访问内存。  
    ResetEvent(g_hEvent);  
}
```

  
可等待的计时器内核对象。

可等待计时器对象，会在某个指定的时间触发或每隔一段时间出发一次。可以调用CreateWaitableTimer创建可等待计时器对象。
```
HANDLE CreateWaitableTimer(
    PSECURITY_ATTRIBUTES psa，  
    BOOL bManualReset,  
    PCTSTR pszName
);
```

  
psa和pszName分别表示安全属性和名称。不再介绍。

bManualReset表示要创建的是一个手动重置计时器还是自动重置计时器。当手动重置计时器被触发，正在等待该计时器的所有线程都变成可调度状态。当自动重置计时器被触发时，只有一个等待该计时器的线程变成可调度状态。

OPenWaitableTimer可以打开一个已经存在的计时器对象并返回该计时器对象的句柄。
```
HANDLE OPenWaitableTimer(
    DWORD dwDesiredAceess,  
    BOOL bInheritHandle,  
    PCTSTR pszName
);
```

  
刚创建的计时器内核对象总是处于未触发状态。当我们想要触发计时器时还需要调用SetWaitableTimer;
```
BOOL SetWaitableTimer(
    HANDLE hTimer,
    Const LARGE_INTEGEr*pDueTime,
    LONG lPeriod,
    PTIMERRAPCROUTING pfnCompletionRoutine,
    PVOID pvArgToCompletionRoutine,
    BOOL bResume
);
```
  
hTimer表示我们要触发的计时器。

pDueTime和lPeriod要配合使用。pDueTime表示计时器第一次触发的时间。lPeriod表示在第一次触发之后，计时器的触发频度，它们都是以毫秒为单位。
```
HANDLE hTimer;  
SYSTEMTIME set;  
FILETIME ftLocal ,ftUTC;  
LARGE_INTEGER liUTC;  
hTimer=CreateWaitableTimer(NULL,false,NULL);  
St.wyear=2013;  
st.wMonth=1;  
st.wDayOfWeek=0;
t.wDay=1;  
st.wHour=1;  
st.wMinute=1;  
1st.wSecond=0;  
1st.wMillisecons=0;  
1SystemTimeToFileTime(&st,&ftLocal);    //SYSTEMTIME结构转换为FILETIME结构。  
1LocalFileTimeToFileTime(&ftLocal,&ftUTC);  
liUTC.LowPart=ftUTC.dwlowDateTime;  
liUTC.HighPart=ftUTC.dwHighDateTime;  
SetWaitableTimer(hTimer,&liUTC,6*60*60*1000,NULL,NULL,false);  
```


上述代码的SYSTEMTIME结构指定第一次触发时间。这个是本地时间。由于SetWaitableTimer的第二个参数是LARGE_INTEGER结构因此不能将它们之间转换。但是FILETIME结构和LARGE_INTEGER结构二进制格式完全相同。我们可以将SYSTEMTIME结构转换为FILETIME结构。由于SetWaitableTimer需要全球标准时间，因此需要调用LocalFileTimeToFileTime完成转换。

由于FILETIME和LARGE_INTEGER二进制格式完全相同。我们更倾向于直接把FILETIME结构传给SetWaitableTimer。

如
```
SetWaitableTimer(hTimer,(PLARGE_INTEGER)&fUTC,6*60**60*1000,NULL,NULL,NULL,false);  
```

  
但是上述代码存在问题。虽然FILETIME和LARGE_INTEGER二进制形式相同，但是它们对其的方式不同。FILETIME结构必须对齐到32位边界。而LARGE_INTEGER结构必须对齐到64位边界。因此直接传递FILTTIME结构可能会出现问题。推荐的方式是先把FILETIME的成员复制到LARGE_INTEGER的成员中，再把LARGE_INTEGER的地址传给SetWaitableTimer。

除了指定一个绝对时间之外，还可以指定一个相对时间。此时只需要将pDuetime传入负值，此时是以100微秒为单位。

下面的代码把计时器第一次触发的时间设为调用结束后的5s后。
```
    HANDLE hTimer;
    LARGE_INTEGER li;
    hTimer=CreateWaitableTimer(NULL,false,NULL);
    Li.quadpart=-(5*10000000);
    SetWaitableTimer(hTimer,&li,6*60*60*1000,NULL,NULL,false);
```
  
此后每隔6小时出发一次。

如果lPeriod传入0，那么计时器只触发一次，以后不再触发。

bResume用以支持挂起。一般都传入false。当传入true时，当计时器被触发时，系统就会使机器结束挂起模式，并唤醒等待该计时器的线程。当为false时，计时器会被触发，但是如果此时机器处于挂起态时，在机器继续执行之前，被唤醒的任何线程都得不到cpu。

最后介绍CancelWaitableTimer
```
BOOL CancelWaitableTimer(HANDLE hTimer);  
```

该函数会将句柄所标识的计时器取消。此后计时器就不会再触发。要想让计时器继续触发，可以再次调用SetWaitableTimer。

用户计时器

相信大家在使用VC时一定接触过用户计时器。它通过SetTimer来设置。但是好多人对它与可等待计时器混淆不清。其实，两者最大区别就是用户计时器需要在用户程序中使用大量的用户界面基础设施，从而消耗更多的资源。而可等待计时器是内核对象，不仅可以在多线程间共享而且具备安全性 。用户计时器会产生WM_TIMER消息，这个消息被送到SetTimer设置的回调函数。此时只有一个线程得到通知。而可等待计时器对象可以被多个线程等待。

如果打算在计时器被触发时执行与用户界面相关的操作。使用用户计时器可使代码更容易编写。

信号量内核对象

信号量内核对象用来对资源进行计数。与其他内核对象相同，它包括一个使用计数。但是它还包含另外两个值：一个是最大资源计数和当前资源计数。最大资源计数表示信号量可以控制的最大资源数量。当前资源计数表示信号量当前可用资源的数量。

假设有个服务器进程，它有一块内存，用可以存储5个客户端请求。如果第六个客户端请求时它将会被拒绝 。此时缓冲区内客户端的请求就是看作资源。我们可以使用信号量来保护资源。首先我们创建信号量并将最大资源计数设为5。由于还没有客户端请求，当前资源计数被初始化为0。随着不断有客户端发出请求，进入请求队列当前资源计数随之递增。随着服务器不断接受客户端请求，请求队列请求减少当前资源使用计数随之递减。

信号量的规则如下：
- 1：如果当前资源计数大于0，那么信号量处于触发状态(说明有资源可被使用，所有等待线程被调度)。
- 2：如果当前资源计数等于0，那么信号量处于未触发状态(没有可用资源，所有线程等待)。
- 3：当前资源计数不会小于0.
- 4：当前可用资源计数不会大于最大资源计数。

注意：使用信号量时不要将信号量的使用计数和当前资源计数混为一谈。

创建信号量对象：
```
HANDLE CreateSomaphore(  
    PSECURITY_ATTRIBUTE psa,  
    LONG lInitialCount,  
    LONG lMaximuCount,  
    PCTSTR pszName
);
```

lMaximumCount告诉系统应用程序能够处理的资源最大数量。

lInitialCount表示这些资源一开始有多少可供使用。

与此对应的有OpenSemaphore。它打开一个已存在的信号量对象
```
HANDLE OpenSemaphore(
    DWORD dwDesiredAccess,
    BOOL hInheritHandle,
    PCTSTR pszName
);
```

为了获得对被保护资源的访问权，线程要调用一个等待函数并传入信号量句柄。在内部等待函数会检查信号量的当前资源计数。如果它的值大于0，那么等待函数会把可用资源数减一并让调度线程继续执行。如果等待函数发现信号量当前资源计数为0，那么系统会让线程进入等待状态。当另一个线程将信号量的当前资源计数递增时，原来由于等待资源被挂起的线程会被唤醒。

线程通过ReleaseSemaphore来递增信号量的当前计数。
```
BOOL ReleaseSemaphore(
    HANDLE hSemaphore,  
    LONG lReleaseCount,  
    PLONG plPreviousCount
);
```

lReleaseCount值被加到当前资源计数上。一般传入1。

plPreviousCount返回加之前的值。这个信号量对象在当前基础上所要增加的值，这个值必须大于0。如果信号量加上这个值会导致信号量的当前值大于信号量创建时指定的最大值，那么这个信号量的当前值不变，同时这个函数返回FALSE。plPreviousCount值不确定。

如果不需要信号量上次的值，那么这个参数可以设置为NULL。

互斥量对象

互斥量内核对象用来确保一个线程独占第一个资源的访问。互斥量对象包括一个使用计数、线程ID以及一个递归计数。

互斥量与关键段行为相同，但是比关键段慢。同时意味着可以在不同进程中访问同一个互斥量。

线程ID用来标识当前占有这个互斥量的是系统中哪个线程。递归计数表示这个线程占用该互斥量的次数。互斥量跟事件内核对象一样被经常使用。

下面是互斥量的使用规则：
- 1：如果线程ID为0，那么该互斥量不为任何线程所占用。处于触发状态。
- 2：如果线程ID非为0，此时有一个线程已经占有了该互斥量，它处于未触发状态。

使用CreateMutex创建一个互斥量。
```
HANDLE CreateMutex(  
    PSECURITY_ATTRIBUTES psa,  
    BOOL bInitialOwner,  
    PCTSTR pszName
);
```

bInitialOwner用来控制互斥量的初始状态。如果传入false，那么表示互斥量不属于任何线程。线程ID和递归计数都为0。此时互斥量处于触发状态。

如果为true，互斥量的线程ID将被设为主调线程的线程ID，递归计数被设为1。

OpenMutex来打开一个已存在的互斥量。
```
HANDLE OpenMutex(  
   DWORD dwDesiredAccess,  
   BOOL bInheritHandle,  
   PCTSTR pszName);  
```

为了获得对被保护资源的访问权，线程要调用等待函数并传入互斥量句柄。在内部，等待函数会检查线程ID是否为0，如果为0，等待线程将互斥量对象线程ID设为当前线程ID，递归计数为1。否则，主调线程将会被挂起。当其他线程完成对保护资源的互斥访问，释放对互斥量的占有时，互斥量的线程ID被设为0，原来被挂起的线程变为可调度状态，并将互斥量对象对象ID设为此线程ID，递归计数为1。

前面一直提到递归计数，却没有解释它的意思。当线程试图等待一个未触发的互斥量对象，此时通常处于等待状态。但是系统会检查想要获得互斥量的线程的线程ID与互斥量对象内部记录的线程ID是否相同。如果相同，那么系统会让线程保持可调度状态，即使该互斥量尚未触发。每次线程等待成功一个互斥量，互斥对象的递归计数就会被设为1。因此，使递归对象大于1 的唯一途径是让线程多次等待同一个互斥量。

当目前占有互斥量的线程不再需要访问互斥资源时，它必须调用ReleaseMutex来释放互斥量。
```
BOOL ReleaseMutex(HANDLE hMutex);  
```


调用ReleaseMutex时该函数会检查调用线程ID是否与互斥量内部保存的线程ID相同。如果相同，那么递归计数会递减。否则函数执行失败返回false。

如果线程成功等待了互斥量对象不止一次，那么线程必须调用相同次数的ReleaseMutex才能使对象的递归计数变成0。

遗弃问题

互斥量具有线程所有权的问题。这就可能导致一些问题：如果占用互斥量的线程在释放互斥量之前终止，对于互斥量来说会发生什么情况呢？答案是：互斥量被遗弃。

系统会检测到互斥量被遗弃的情况，会自动的将互斥量对象的线程ID置为0，然后再检查有没有其他线程等待该互斥量。如果有线程等待，系统会从等待队列中选取一个，将其变为可调度状态。这一切都和原来没什么不同，唯一的区别是等待函数不再返回WAIT_OBJEC_0，而是返回WAIT_ABANDONED。这个特殊的返回值只适用于互斥量。

看例子：
该例展示了使用信号量和互斥量来实现生产者和消费者问题。定义一个缓冲区，互斥量负责使各线程互斥访问缓冲区。信号量代表已生产的产品的个数。

其实这并不是一个完整的消费者和生产者问题。完整的消费者生产者问题需要定义两个信号量和一个互斥量。互斥量负责各线程互斥访问缓冲区，两个信号量分别监视当前已生产产品数和缓冲区剩余空间。当有已生产产品时通知消费者取。当缓冲区未满时通知生产者生产。而此例仅仅定义了一个信号量用以监视已生产产品数量，当有产品时通知消费者，但是在缓冲区满时并未通知生产者等待。仅仅是判断下，让生产者打出一行提示“缓冲区已满”并没有让生产者等待。

在生产者进入循环后，仅仅判断mutex，并不判断是否有空余缓冲区。而消费者在进入循环后，不仅判断mutex也判断是否有产品。这是与经典的生产者与消费者问题的区别。需要特别注意！！

也要注意ReleaseSemaphore的使用。当信号量增加后的值大于最大值，则函数返回false。返回的原来的值不确定。

CQueue头文件
```
#pragma once  
typedef struct ELEMENT  
{  
    DWORD tid;  
    int num;  
    //CMutexAndSemaphoreTestDlg *pDlg;  
}*PELEMENT;

class CQueue  
{
public:
    CQueue(void);
    ~CQueue(void);  

public:
    bool QueueIn(ELEMENT*p);
    PELEMENT QueueOut();  

public:  
    HANDLE m_hSemaphore;
    HANDLE m_hMutext;
    HANDLE m_array[2];
    PELEMENT m_pelement[10];
    int m_curIndex;  
};

CQueue::CQueue(void)  
{
    m_hMutext=CreateMutex(NULL,false,NULL);  
    m_hSemaphore=CreateSemaphore(NULL,0,10,NULL);  
    //m_pelement=new PELEMENT[10];  
    m_curIndex=0;  
    m_array[0]=m_hSemaphore; //先判断同步，再判断互斥。但是书上并没有这样，不知道WaitForMultipleOBjecs会怎样处理？20121230    
    m_array[1]=m_hMutext; 
    //查了下msdn,介绍如下
    //When bWaitAll is TRUE, the function's wait operation is completed   
    //only when the states of all objects have been set to signaled.
    //The function does not modify
    //the states of the specified objects until the states of all objects have been set to   
    //signaled. For example, a mutex can be signaled, but the thread does not get ownership   
    //until the states of the other objects are also set to signaled. In the meantime,   
    //some other thread may get ownership of the mutex, thereby setting its state to nonsignaled.
}
  
//出队。
PELEMENT CQueue::QueueOut()  
{
    PELEMENT p;  
    p=m_pelement[0];  
    for(int i=0;i<m_curIndex-1;i++)
    {
        m_pelement[i]=m_pelement[i+1];
    }  
    m_curIndex--;
    ReleaseMutex(m_hMutext);
    return p;
}

//入队

bool CQueue::QueueIn( ELEMENT*p )  
{  
    long pp;  
    bool ret=ReleaseSemaphore(m_hSemaphore,1,&pp);  
    if(ret==false) //增加大最大值不再增加。ret返回false.pp返回任一值。  
    {
        ReaseMutex(m_hMutext);
        return false;  
    }
    m_pelement[m_curIndex]=p;  
    m_curIndex++;  
    ReleaseMutex(m_hMutext);  
    return true;
}  
```

消费者线程和生产者线程
```
DWORD WINAPI CMutexAndSemaphoreTestDlg::ServerThread( PVOID pparam )  
{  
    CMutexAndSemaphoreTestDlg *p=(CMutexAndSemaphoreTestDlg*)pparam;  
    while(true)  
    {
        WaitForMultipleObjects(2,p->queue.m_array,TRUE,INFINITE);
        InterlockedIncrement(&m);
        PELEMENT q=p->queue.QueueOut();
        CString s;
        s.Format(TEXT("服务线程[%d]第%d次取出产品"),GetCurrentThreadId(),m);  
        p->m_ServerList.InsertString(-1,s);
    }
}

DWORD WINAPI CMutexAndSemaphoreTestDlg::ClientThread( PVOID pparam )  
{
    //int m=1;  
    CMutexAndSemaphoreTestDlg *p=(CMutexAndSemaphoreTestDlg*)pparam;  
    while(1)
    {
        WaitForSingleObject(p->queue.m_hMutext,INFINITE);
        //WaitForMultipleObjects(2,p->queue.m_array,TRUE,INFINITE);  
        ELEMENT *q=new ELEMENT;  
        q->tid=GetCurrentThreadId();  
        bool ret=p->queue.QueueIn(q);  
        CString s;  
        if(ret)  
        {
            InterlockedIncrement(&n);
            s.Format(TEXT("客户线程[%d]第%d次添加!"),q->tid,n);  
        }
        else  
        {  
            s.Format(TEXT("队列已满！！"));  
        }  
        p->m_ClientList.InsertString(-1,s);  
        Sleep(1000);  
    }  
    return true;  
}  

//创建生产者和消费者线程。
void CMutexAndSemaphoreTestDlg::OnBnClickedOk()  
{  
    // TODO: 在此添加控件通知处理程序代码  
    m_hClientThread=CreateThread(NULL,0,ClientThread,this,0,NULL);  
    m_hClientThread=CreateThread(NULL,0,ClientThread,this,0,NULL); 
    //m_hServerThread=CreateThread(NULL,0,ServerThread,this,0,NULL);  
    m_hServerThread=CreateThread(NULL,0,ServerThread,this,0,NULL);  
    //CDialogEx::OnOK();  
}  
```