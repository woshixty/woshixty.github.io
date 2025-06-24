转载自: [《windows核心编程系列》四 谈谈进程的建立和终止](https://www.cnblogs.com/weekbo/p/9054488.html)

# 《windows核心编程系列》四 谈谈进程的建立和终止

上一章介绍了内核对象，这一节开始就要不断接触各种内核对象了。首先要给大家介绍的是进程内核对象。进程大家都不陌生，它是资源和分配的基本单位，而进程内核对象就是与进程相关联的一个数据结构。操作系统内核通过它管理进程，也就是操作系统原理上介绍的进程控制块（PCB）。举个例子，它就相当于每个学生都有的学籍，学校管理我们都是通过学籍，什么记过了，处分了，开除学籍了，都是在学籍上做文章。

进程一般被定义为一个正在运行的程序的一个实例，它由两部分组成：
- 1：内核对象，操作系统用它来管理进程。内核对象也是系统保存进程统计信息的地方。
- 2：一个地址空间，其中包含所有可执行文件或DLL模块的代码和数据。

Windows支持两种类型的应用程序：GUI程序和CUI程序。前者是我们经常接触的，具有窗口外观的窗口应用程序。后者是控制台应用程序。在使用vc来开发应用程序时，会设置各种链接器开关。链接器根据这些开关将子系统的正确类型嵌入最终生成的可执行文件。对于CUI程序这个开关是/SUBSYSTEM:CONSOLE。对于GUI程序，则是/SUBSYSTEM:WINDOWS。

这些开关会告诉链接器在链接时链接什么入口函数。对于GUI程序它的入口点函数时WinMain，CUI程序是main。

有人以为入口函数就是程序执行的开始，其实这是不正确的。在入口点函数之前还有一个被称为启动函数的函数。该函数用来初始化C/C++运行库、构造全局和静态的C++对象等。

根据应用程序类型的不同，启动函数也不一样。ANSI字符集下，GUI程序的启动函数是WinMainCRTStartup，入口函数是WinMain。CUI的启动函数是mainCRTStartup，入口函数是main。Unicode字符集下，GUI程序的启动函数是wWinMainCRTStartup，入口函数是wWinMain，CUI的启动函数是wmainCRTStartup，入口点函数时wmain。

我们在写控制台下的应用程序时，可以通过argv来引用命令行参数，当时也很疑惑，为什么可以直接用呢？原来都是启动函数的功劳。它会在进入入口函数之前帮我们做其他工作：

- 1：获取命令行指针。
- 2：获取指向环境变量的指针
- 3：初始化C/C++运行库的全局变量。
- 4：初始化C运行库内存分配函数。
- 5：调用所有全局和静态C++类对象的构造函数。

完成所有这些工作后，启动函数就会调用应用程序的入口点函数。入口点函数返回后启动函数获得入口点函数返回值，并将其传递给C运行库函数exit。Exit函数将调用所有全局和静态C++类对象的析构函数和其他清理工作。然后将入口函数的返回值传递给ExitProcess函数，结束进程并设置返回值为退出代码。

加载到进程地址空间的可执行文件或是DLL都有一个实例句柄。用以标识它在进程地址空间的位置。可执行文件的实例句柄被当做WinMain函数的第一个参数传入。它实际上是一个内存基地址。系统将可执行文件的映像加载到进程地址空间中的这个位置。映像加载到哪个地址是由链接器决定的。不同的链接器使用不同的首先基地址。exe文件和dll都会有一个默认的首选基地址。exe文件是400000，dll是10000000。

为了获得一个可执行文件或dll文件被加载到进程地址空间的位置，可以使用GetModuleHandle函数。它需要一个以/0结尾标示可执行文件或dll的名字字符串为参数。当传入NULL时，此时将会返回主调进程可执行文件的基地址，即使此时代码在一个dll文件中仍然是这样。如果此时代码在dll中执行，我们想何知道此时代码正在什么模块中运行，这可以通过GetModuleHandleEx得到。将GET\_MODULE\_HANDLE\_EX\_FLAG\_FROM\_ADDRESS作为它的第一个参数，再将当前函数的地址作为它的第二个参数，函数执行完毕，最后一个参数将保存出入的函数所在dll的基地址。

系统在创建进程时会传给他一个命令行，这个命令行总是非空，因为它至少存储有可执行文件的名称。C运行库的启动代码在执行一个GUI应用程序时，会调用windows函数GetCommandLine来获得进程的完整命令行，它忽略可执行文件名称，然后将剩余部分传给WinMain的pszCmdLine参数。

每个进程都有一个与它相关联的环境块。用以定义工作环境、保存有用信息，使系统获得相关设置。应用程序经常利用环境变量让用户精细其行为。用户创建一个环境变量并进行初始化，此后应用程序运行时会正在环境块中查找变量，如果找到变量就会解析变量的值，并调整自己的行为。它所占用的内存是在进程地址空间内分配的。同样调用GetEnvironmentStrings函数可以获得完整的环境块。通常子进程会继承一组环境变量，这些环境变量和父进程的环境变量相同，父进程可以控制那些环境变量允许子进程继承。注意子进程继承的仅仅是父进程环境变量的副本，它们不共享同一个环境块。GetEnvironmentVariable函数可以用来判断一个环境变量是否存在。

在下图中我们可以看到形如%USERPROFILE%的字符串，它表示两个%之间的这部分内容是一个可替换的变量。该变量在环境变量中已经被定义。

可以使用SetEnvironmentVariable来添加、删除或修改一个变量。

Windows不建议使用入口函数的参数来访问命令行或是环境变量，而应该使用以上介绍的各种函数。应该将它们当做只读变量，不要对它们进行修改。

在多处理器的系统中，可以强迫线程在某个cpu上运行，这成为处理器关联性。子进程继承了其父进程的关联性。

如果不提供完整的路径名，windows函数就会在当前驱动器的当前目录查找文件和目录。如：调用CreateFile打开一个文件时，如果仅指定文件名，系统将在当前驱动器和目录查找该文件。

系统在内部跟踪记录这一个进程当前驱动器和目录，这些信息是以进程为单位来维护的，如果该进程的一个线程更改了当前驱动器和目录，则只影响本进程的所有线程。

一个线程可以使用GetCurrentDirectory和SetCurrentDirectory来获得和设置当前驱动器和目录。子进程的当前目录默认为每个驱动器的根目录。如果父进程希望子进程继承它的当前目录，就必须在生成子进程之前，添加环境变量。

使用GetVersionEx可以获得window系统的版本号。

`CreateProcess`函数，接下来进入到本章最重要的知识点：CreateProcess函数。

该函数用以创建一个进程：

```
Bool CreateProcess(  
    PCTSTR pszApplicationName,
    PTSTR pszCommandLine,
    PSECURTITY\_ATTRIBUTES psaProcess,
    PSECUTRITY\_ATTRIBUTE psaThread,
    Bool hInheritHandles,
    DWROD fdwCreate,
    PVOID pvEnvironment,
    PCTSTR pszCurDir,
    PSTARTUPINFO psiStartInfo,
    PPROCESS\_INFORMATION ppiProcInfo
);  
```

当此函数被调用时，系统将首先创建一个进程对象，其初始使用计数为1。进程内核对象并不是进程本身，而是操作系统用来管理这个进程的一个数据结构。此后系统为新进程创建一个虚拟地址空间，并将可执行文件的代码及数据加载到进程的地址空间。然后系统会新进程的主线程创建一个线程内核对象，也将其使用计数设为1。和进程内核对象一样它也是操作系统用以管理线程的数据结构。主线程首先会执行C/C++运行时的启动函数，启动函数调用入口函数。进程被创建成功后CreateProcess返回true，函数返回前CreateProcess可能还没有完全初始化好。

psaApplicationName和pszCommandLine分别指定新进程要使用的可执行文件称和要传给新进程的命令行字符串。

注意此处的pszCommandLine是非常量字符串。传入常量字符串将会导致访问违规，因为在内部CreateProcess会修改传入的命令行字符串，返回时再将这个字符串还原。

所以一下代码是错误的：
```
STARTUPINFO si = {sizeof(si)};
PROCESS\_INFORMATION pi;
CreateProcess(NULL, TEXT(“NOTEPAD”), NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi);
```

因为TEXT（"NOTEPAD"）是常量字符串，当CreateProcess试图修改字符串会引起访问违规。解决方法是将TEXT（"NOTEPAD"）放在一个缓冲区内：
```
STARTUPINFO si={sizeof(si)};  
PROCESS\_INFORMATION pi;  
TCHAR cmdLine\[200\]=TEXT("NOTEPAD");  
CreateProcess(NULL,cmdLine,NULL,NULL,FALSE,0,NULL,NULL,&si,&pi);  
```

这一点要特别注意，很容易出错！！！！这也有个例外，windows vista以及win7的ANSI版本以上是不会发生访问违规的，因为它们会为命令行创建一个临时副本。

在解析命令行时，CreateProcess会检查字符串中第一个标记，假定此标记就是我们想运行的可执行文件名称。如果可执行文件没有扩展名，就会默认是.exe扩展名。CreateProcess会在以下目录下搜索可执行文件：

1：主调进程.exe文件所在目录。

2：主调进程的当前目录。

3：windows系统目录。即System32目录。

4：windows目录。

5：PATH环境变量中列出的目录。

如果为可执行文件制定完整路径，系统就会按指定路径寻找。

以上情况在pszApplicationName为NULL时才发生。当然也可以在pszApplicationName传递可执行文件名称，此时必须指定扩展名，否则进程不会被创建。系统会按照此处指定的路径寻找，如没有指定完整路径，系统会假定文件位于当前目录。如找不到，函数调用失败。

当pszApplicationName指定文件名，pszCommandLine参数中的内容也会作为新进程的命令行传给它。如：
```
STARTUPINFO si={sizeof(si)};  
PROCESS\_INFORMATION pi;  
TCHAR cmdLine\[200\]=TEXT("WORDPAD a.txt");  
CreateProcess(TEXT("C:\\\\windows\\\\system32\\\\NOTEPAD.exe"),  
cmdLine,NULL,NULL,FALSE,0,NULL,NULL,&si,&pi);  
```

命令行是“WORDPAD a.txt”，记事本程序将询问：a.txt不存在是否创建a.txt文件。第一个参数WORDPAD应该是作为程序名称传入的。

为了创建一个新的进程，系统必须创建一个进程内核对象和一个线程内核对象，由于这些都是内核对象，所以父进程必须将安全属性关联到这两个对象上。可以通过psaProcess和psaThread为进程和线程安全对象指定默认的安全描述符。可以都为它们指定NULL，使用默认的安全属性，也可以分配并初始化两个SECURITY\_ATTRIBUTES结构，以便创建安全权限并将它们分配给进程和线程对象。

fdwCreate参数影响新进程创建的方式。如：指定CREATE\_SUSPENDEd标识让系统在创建新进程时挂起其主线程。这样父进程就可以修改子进程地址空间的内存、更改主线程优先级或是将其添加到作业中。修改完后可以调用ResumeThread来允许子进程执行代码。传入0 表示创建进程后立即运行。多个标志位可以组合使用。

pvEnvironment参数指定一块内存，包含新进程要使用的环境字符串。但多数情况下都是传入NULL，表示子进程要继承父进程使用的一组环境字符串。也可以使用GetEnvironmentStrings函数，此函数获取主调进程正在使用的环境字符串地址，当我们为pvEnvirtonment传入NULL时，CreateProcess就是调用这个函数。不使用这块内存时应该使用FreeEnvironmentStrings函数。

pszCurDir允许父进程设置子进程的当前驱动器和目录。如果为NULL，新进程的工作目录与父进程的一样。

psiStartInfo参数指向一个STARTUPINFO结构。该结构包含很多成员。Windows在创建新进程的时候使用它们，但是大多数应用程序仅仅使用它们的默认值。因此我们要做的最起码的工作就是将此结构的所有成员都初始化为0，将cb成员设为此结构的大小，如：

STARTUPINFO si={sizeof(si)};

此时除cb成员外，其他成员均为0。不能仅仅si.cb=sizeof(si);因为此时其他成员的值都是垃圾数据。

ppiProcInfo参数指向PROCESS\_INFORMATION结构，CreateProcess在返回时会初始化这个结构。
```
Typedef struct \_PROCESS\_INFORMATION
{
   HANDLE hProcess;
   HANDLE hThread;
   HANDLE dwProcessId;
   HANDLE dwThreadId;
} PROCESS\_INFORMATION;
```

CreateProcess创建的进程和线程对象将通过它返回。创建时系统会为每个对象指定一个初始的使用计数1，CreateProcess返回时由于PROCESS\_INFORMATION结构中再次引用了进程和线程内核对象 此时它们的使用计数都变为2。可以理解为进程和线程实例本身也占有一个计数。当它们结束运行时这个使用计数被递减1。

此时如果系统要释放进程对象，1：进程必须终止，此时使用计数递减1。2：父进程必须调用CloseHandle，使用计数再次减去1。线程类似。

因此为了使不再使用的内核对象能够得到释放，一定要在不使用时调用CloseHandle关闭对句柄的引用。所有对该句柄的引用都被关闭后，当进程或线程终止时它们关联度的内核对象才能够被释放。

CreateProcess还会为进程和线程分配一个ID号。进程和线程分享同一个号码池。这意味着它们不可能相同。一个对象的ID不可能分配到0，因为windows任务管理器将进程ID 0与系统空闲进程关联。该进程代表未被真实使用的cpu使用率。

dwProcessId和dwThreadId成员就是存储进程和线程的ID。使用GetCurrentProcessId可以获得当前进程的ID。GetCurrentThreadId来获得当前正在运行的线程的ID。另外使用GetProcessId和GetThreadId可以获得指定句柄对应的进程和线程的ID。使用GetProcessIDOfThread可以获得某句柄关联的线程所在进程的ID。

由于ID可能会立即重用。也就是说当我们获得某个进程的ID并保存后，此后在使用时有可能出现它已经被释放了。此时此ID就对应着其他进程了。避免这种情况的唯一方法就是：保证进程或线程对象不被销毁。

进程终止

进程可以通过三种方式终止：

1：主线程从入口函数返回。

2：进程中的一个线程调用ExitProcess。

3：另一个进程中的线程调用TerminateProcess。

在以上介绍的三种方式中仅有第一种，当主线程从入口函数返回才保证主线程的所有资源都会被正确清理。

清理操作包括：

1：调用所有在本进程内使用的任何C++对象的析构函数。

2：释放各个线程线程栈使用的内存。

3：进程的退出代码被设为入口函数的返回值。

4：进程内核对象使用计数递减1。

   正常情况下入口点函数会返回到启动函数，启动函数将正确清理进程使用的所有C运行时资源，清理之后启动代码显式调用ExitProcess并将入口函数返回值传给它。这也是为什么只需从入口函数返回却可以终止整个进程的原因。

进程的一个线程调用ExitProcess可以终止本进程。其后的别的代码将不会被执行。

与ExitProcess相类似的还有ExitThread，它会导致一个线程终止。在创建线程时常出现这种情况：子线程还没有怎么执行程序就已经结束了，这有可能是在创建完线程后，主线程没有调用WaitForSingleObject之类的函数，主线程创建完其他线程后就返回到启动函数函数返回整个进程被终止。这一点很容易出错。

调用ExitProcess或是ExitThread会导致进程或线程当场终止运行，再也不会返回到启动函数，清理工作（C++对象的析构）当然没法执行。虽然最终随着进程的结束，该进程内所有线程所使用的资源都会被释放，但是应该避免调用这些函数，它们阻止了C++对象析构函数对善后工作的处理。顺便提下，如果在主线程调用ExitThread，虽然主线程当场终止，但是如果进程内还有其他线程，则进程不会终止。

如以下代码：
```
#include<windows.h>
#include<iostream>
DWORD ThreadProc(PVOID)
{
    int i=0;
    int j=0;
    while(i<1000000)
    {
        i++;
        while(j<10000) j++;
        std::cout<< i << "," << j << std::endl;
    }
    return 0;
}

int main(int argc,char\*\*argv)
{
    DWORD id;
    CreateThread(NULL,0,(LPTHREAD\_START\_ROUTINE)ThreadProc,NULL,0,&id);
    ExitThread(0);
    return 0;
}
```

ExitProcess,ExitThread只能由本进程的其他线程调用。而TerminateProcess和TerminateThread却可以由任何其他进程的线程调用。它的第一个参数指定要终止进程的句柄。这种情况下应用程序得不到自己要被杀死的通知，也不能阻止自己被杀死，当然也无法为自己准备好后事（得不到清理）。例如已经修改的文件没有刷新到磁盘上。

但要明白进程终止后属于它的任何东西都会被释放。TerminateProcess是异步的，此函数调用后我们并不能保证进程已经被强行终止了。要确定进程是否终止可以调用WaitForSingleObject函数，并传入进程句柄。

进程终止时进行的操作。

一个进程终止时，系统会依次执行以下操作：

```
1：终止进程中遗留的任何线程。
2：释放进程分配的所有用户对象，关闭所有内核对象。如果它们的使用计数变为0，内核对象将会释放。
3: 将进程的退出代码从STILL\_ACTIVE变为传给ExitProcess或是TerminateProcess的参数存储在内核对象中。
4：进程内核对象变为一触发状态。这也是为什么其他线程可以挂起他们自己直至另一个进程终止运行。
5：进程内核对象的使用计数递减1。
```

进程内核对象的生命期至少能像进程本身一样长。当进程终止时如果系统中还有另一个进程打开了这个进程的内核对象的句柄，进程内核对象的使用计数就不会变为0。当父进程忘记关闭子进程的句柄时往往发生这种情况。

进程终止了内核对象还没有被释放，这样做有用吗？当然有用！！即使进程终止了，存储在内核对象的信息也有可能被使用，如我们想知道进程占用了多少Cpu时间，或是想获得它的退出代码。GetExitCodeProcess此函数会查找进程内核对象并从内核对象的数据结构中取出退出代码。任何时候都可以调用此函数。如此时进程正在运行那么将会得到STILL\_ALIVE。

WaitForSingleObject将会挂起当前线程，知道它所等待的对象变为已触发状态。进程或线程对象在终止时就会变成已触发状态。
