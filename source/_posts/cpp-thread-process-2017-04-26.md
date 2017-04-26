---
title: C++之线程与进程
date: 2017-04-26 10:47:37
tags: [C/C++]
categories: 编程
---
### 进程与线程的概念
1. 任务调度
大部分操作系统(如Windows、Linux)的任务调度是采用时间片轮转的抢占式调度方式，即一个任务执行一小段时间后强制暂停去执行下一个任务，每个任务轮流执行。任务执行的一小段时间叫做时间片，任务正在执行时的状态叫运行状态，任务执行一段时间后强制暂停去执行下一个任务，被暂停的任务就处于就绪状态等待下一个属于它的时间片的到来,这样每个任务都能得到执行.

2. 进程
操作系统是计算机的管理者，它负责任务的调度、资源的分配和管理，统领整个计算机硬件；应用程序则是具有某种功能的程序，程序是运行于操作系统之上的.
在早期的操作系统中并没有线程的概念，进程是能拥有资源和独立运行的最小单位，也是程序执行的最小单位。任务调度采用的是时间片轮转的抢占式调度方式，而进程是任务调度的最小单位，每个进程有各自独立的一块内存，使得各个进程之间内存地址相互隔离。

进程是一个具有一定独立功能的程序在一个数据集上的一次动态执行的过程，是操作系统进行资源分配和调度的一个独立单位，是应用程序运行的载体。

进程是一种抽象的概念，从来没有统一的标准定义。进程一般由程序、数据集合和进程控制块三部分组成。
程序用于描述进程要完成的功能，是控制进程执行的指令集；
数据集合是程序在执行时所需要的数据和工作区；
程序控制块(Program Control Block，简称PCB)，包含进程的描述信息和控制信息，是进程存在的唯一标志。

进程具有的特征：
动态性：进程是程序的一次执行过程，是临时的，有生命期的，是动态产生，动态消亡的；
并发性：任何进程都可以同其他进程一起并发执行；
独立性：进程是系统进行资源分配和调度的一个独立单位；
结构性：进程由程序、数据和进程控制块三部分组成。

3. 线程
线程是程序执行中一个单一的顺序控制流程，是程序执行流的最小单元，是处理器调度和分派的基本单位。
一个进程可以有一个或多个线程，各个线程之间共享程序的内存空间(也就是所在进程的内存空间)。
一个标准的线程由线程ID、当前指令指针(PC)、寄存器和堆栈组成,
而进程由内存空间(代码、数据、进程空间、打开的文件)和一个或多个线程组成。

### 进程与线程的区别
1. 线程是程序执行的最小单位，而进程是操作系统分配资源的最小单位；
2. 一个进程由一个或多个线程组成，线程是一个进程中代码的不同执行路线；
3. 进程之间相互独立，但同一进程下的各个线程之间共享程序的内存空间(包括代码段、数据集、堆等)及一些进程级的资源(如打开文件和信号)，某进程内的线程在其它进程不可见；
4. 调度和切换：线程上下文切换比进程上下文切换要快得多。
总之，线程和进程都是一种抽象的概念，线程是一种比进程更小的抽象，线程和进程都可用于实现并发。
在早期的操作系统中并没有线程的概念，进程是能拥有资源和独立运行的最小单位，也是程序执行的最小单位。它相当于一个进程里只有一个线程，进程本身就是线程。所以线程有时被称为轻量级进程(Lightweight Process，LWP）。
后来，随着计算机的发展，对多个任务之间上下文切换的效率要求越来越高，就抽象出一个更小的概念——线程，一般一个进程会有多个(也可是一个)线程。

### 多线程与多核
很多操作系统的书都说“同一时间点只有一个任务在执行”,其实“同一时间点只有一个任务在执行”这句话是不准确的，至少它是不全面的。那多核处理器的情况下，线程是怎样执行呢？这就需要了解内核线程。
多核(心)处理器是指在一个处理器上集成多个运算核心从而提高计算能力，也就是有多个真正并行计算的处理核心，每一个处理核心对应一个内核线程。
内核线程（Kernel Thread， KLT）就是直接由操作系统内核支持的线程，这种线程由内核来完成线程切换，内核通过操作调度器对线程进行调度，并负责将线程的任务映射到各个处理器上。一般一个处理核心对应一个内核线程，比如单核处理器对应一个内核线程，双核处理器对应两个内核线程，四核处理器对应四个内核线程。

现在的电脑一般是双核四线程、四核八线程，是采用超线程技术将一个物理处理核心模拟成两个逻辑处理核心，对应两个内核线程，所以在操作系统中看到的CPU数量是实际物理CPU数量的两倍，如你的电脑是双核四线程，打开“任务管理器\性能”可以看到4个CPU的监视器，四核八线程可以看到8个CPU的监视器。

超线程技术就是利用特殊的硬件指令，把一个物理芯片模拟成两个逻辑处理核心，让单个处理器都能使用线程级并行计算，进而兼容多线程操作系统和软件，减少了CPU的闲置时间，提高的CPU的运行效率。这种超线程技术(如双核四线程)由处理器硬件的决定，同时也需要操作系统的支持才能在计算机中表现出来。

程序一般不会直接去使用内核线程，而是去使用内核线程的一种高级接口——轻量级进程（Light Weight Process，LWP），轻量级进程就是我们通常意义上所讲的线程(我们在这称它为用户线程)，由于每个轻量级进程都由一个内核线程支持，因此只有先支持内核线程，才能有轻量级进程。用户线程与内核线程的对应关系有三种模型：一对一模型、多对一模型、多对多模型.
- 一对一模型
一个用户线程就唯一地对应一个内核线程(反过来不一定成立，一个内核线程不一定有对应的用户线程)。这样，如果CPU没有采用超线程技术(如四核四线程的计算机)，一个用户线程就唯一地映射到一个物理CPU的线程，线程之间的并发是真正的并发。一对一模型使用户线程具有与内核线程一样的优点，一个线程因某种原因阻塞时其他线程的执行不受影响；此处，一对一模型也可以让多线程程序在多处理器的系统上有更好的表现。
但一对一模型也有两个缺点：
1.许多操作系统限制了内核线程的数量，因此一对一模型会使用户线程的数量受到限制；
2.许多操作系统内核线程调度时，上下文切换的开销较大，导致用户线程的执行效率下降。
- 多对一模型
多对一模型将多个用户线程映射到一个内核线程上，线程之间的切换由用户态的代码来进行，因此相对一对一模型，多对一模型的线程切换速度要快许多；此外，多对一模型对用户线程的数量几乎无限制。
但多对一模型也有两个缺点：
1.如果其中一个用户线程阻塞，那么其它所有线程都将无法执行，因为此时内核线程也随之阻塞了；
2.在多处理器系统上，处理器数量的增加对多对一模型的线程性能不会有明显的增加，因为所有的用户线程都映射到一个处理器上了。
- 多对多模型
多对多模型结合了一对一模型和多对一模型的优点，将多个用户线程映射到多个内核线程上。
多对多模型的优点有：
1.一个用户线程的阻塞不会导致所有线程的阻塞，因为此时还有别的内核线程被调度来执行；
2.多对多模型对用户线程的数量没有限制；
3.在多处理器的操作系统中，多对多模型的线程也能得到一定的性能提升，但提升的幅度不如一对一模型的高。
在现在流行的操作系统中，大都采用多对多的模型。

### 进程和线程的状态
当线程的数量小于处理器的数量时，线程的并发是真正的并发，不同的线程运行在不同的处理器上。但当线程的数量大于处理器的数量时，线程的并发会受到一些阻碍，此时并不是真正的并发，因为此时至少有一个处理器会运行多个线程。
在单个处理器运行多个线程时，并发是一种模拟出来的状态。操作系统采用时间片轮转的方式轮流执行每一个线程。现在，几乎所有的现代操作系统采用的都是时间片轮转的抢占式调度方式，如我们熟悉的Unix、Linux、Windows及Mac OS X等流行的操作系统。
线程是程序执行的最小单位，也是任务执行的最小单位。在早期只有进程的操作系统中，进程有五种状态，创建、就绪、运行、阻塞(等待)、退出。早期的进程相当于现在的只有单个线程的进程，那么现在的多线程也有五种状态，现在的多线程的生命周期与早期进程的生命周期类似。

- 进程的状态
进程在运行过程有三种状态：就绪、运行、阻塞，创建和退出状态描述的是进程的创建过程和退出过程。
创建：进程正在创建，还不能运行。操作系统在创建进程时要进行的工作包括分配和建立进程控制块表项、建立资源表格并分配资源、加载程序并建立地址空间；
就绪：时间片已用完，此线程被强制暂停，等待下一个属于他的时间片到来；
运行：此线程正在执行，正在占用时间片；
阻塞：也叫等待状态，等待某一事件(如IO或另一个线程)执行完；
退出：进程已结束，所以也称结束状态，释放操作系统分配的资源。
- 线程的状态
创建：一个新的线程被创建，等待该线程被调用执行；
就绪：时间片已用完，此线程被强制暂停，等待下一个属于他的时间片到来；
运行：此线程正在执行，正在占用时间片；
阻塞：也叫等待状态，等待某一事件(如IO或另一个线程)执行完；
退出：一个线程完成任务或者其他终止条件发生，该线程终止进入退出状态，退出状态释放该线程所分配的资源。

### 线程优先级
优先级调度(Priority Schedule)决定了线程按照什么顺序轮流执行.线程拥有各自的线程优先级(Thread Priority),线程的优先级可以由用户手动设置，此外系统也会根据不同情形调整优先级。频繁等待的线程称之为IO密集型线程(IO Bound Thread)，而把很少等待的线程称之为CPU密集型线程(CPU Bound Thread)。IO密集型线程总是比CPU密集型线程更容易得到优先级的提升。

线程饿死:优先级较低的线程，在它执行之前总是有比它优先级更高的线程等待执行，因此这个低优先级的线程始终得不到执行。

在优先级调度环境下，线程优先级的改变有三种方式： 
1. 用户指定优先级； 
2. 根据进入等待状态的频繁程度提升或降低优先级(由操作系统完成)； 
3. 长时间得不到执行而被提升优先级。

### 线程安全与锁
多个线程对同一数据的进行访问时需要同步，以确保线程安全。
同步(synchronization)就是指一个线程访问数据时，其它线程不得对同一个数据进行访问，即同一时刻只能有一个线程访问该数据，当这一线程访问结束时其它线程才能对这它进行访问。
同步最常见的方式就是使用锁(Lock)，也称为线程锁。锁是一种非强制机制，每一个线程在访问数据或资源之前，首先试图获取(Acquire)锁，并在访问结束之后释放(Release)锁。在锁被占用时试图获取锁，线程会进入等待状态，直到锁被释放再次变为可用。

1. 二元信号量
二元信号量(Binary Semaphore)是一种最简单的锁，它有两种状态：占用和非占用。它适合只能被唯一一个线程独占访问的资源。当二元信号量处于非占用状态时，第一个试图获取该二元信号量锁的线程会获得该锁，并将二元信号量锁置为占用状态，之后其它试图获取该二元信号量的线程会进入等待状态，直到该锁被释放。

2. 信号量
多元信号量允许多个线程访问同一个资源，多元信号量简称信号量(Semaphore)，对于允许多个线程并发访问的资源，这是一个很好的选择。一个初始值为N的信号量允许N个线程并发访问。线程访问资源时首先获取信号量锁，进行如下操作： 
1). 将信号量的值减1； 
2). 如果信号量的值小于0，则进入等待状态，否则继续执行； 
访问资源结束之后，线程释放信号量锁，进行如下操作： 
1). 将信号量的值加1； 
2). 如果信号量的值小于1(等于0)，唤醒一个等待中的线程；

3. 互斥量
互斥量(Mutex)和二元信号量类似，资源仅允许一个线程访问。与二元信号量不同的是，信号量在整个系统中可以被任意线程获取和释放，也就是说，同一个信号量可以由一个线程获取而由另一线程释放。而互斥量则要求哪个线程获取了该互斥量锁就由哪个线程释放，其它线程越俎代庖释放互斥量是无效的。

4. 临界区
临界区(Critical Section)是一种比互斥量更加严格的同步手段。互斥量和信号量在系统的任何进程都是可见的，也就是说一个进程创建了一个互斥量或信号量，另一进程试图获取该锁是合法的。而临界区的作用范围仅限于本进程，其它的进程无法获取该锁。除此之处，临界区与互斥量的性质相同。

5. 读写锁
读写锁(Read-Write Lock)允许多个线程同时对同一个数据进行读操作，而只允许一个线程进行写操作。这是因为读操作不会改变数据的内容，是安全的；而写操作会改变数据的内容，是不安全的。对同一个读写锁，有两种获取方式：共享的(Shared)和独占的(Exclusive)。当锁处于自由状态时，试图以任何一种方式获取锁都能成功，并将锁置为对应的状态；如果锁处于共享状态，其它线程以共享方式获取该锁，仍然能成功，此时该锁分配给了多个线程；如果其它线程试图如独占的方式获取处于共享状态的锁，它必须等待所有线程释放该锁；处于独占状态的锁阻止任何线程获取该锁，不论它们以何种方式。获取读写锁的方式总结如下：
|读写锁的状态	|以共享方式获取	|以独占方式获取|
|------------|---------------|-------------|
|自由	|成功	|成功|
|共享	|成功	|等待|
|独占	|等待	|等待|

### 单线程
任何程序至少有一个线程，即使你没有主动地创建线程，程序从一开始执行就有一个默认的线程，被称为主线程(main thread)，只有一个线程的程序称为单线程程序。

### 线程使用

1. 创建线程
在Windows平台，Windows API提供了对多线程的支持。Windows中线程相关的操作和方法：`CreateThread与CloseHandle`.
CreateThread用于创建一个线程，其函数原型如下：

```cpp
HANDLE WINAPI CreateThread(
    LPSECURITY_ATTRIBUTES   lpThreadAttributes, //线程安全相关的属性，常置为NULL
    SIZE_T                  dwStackSize,        //新线程的初始化栈在大小，可设置为0
    LPTHREAD_START_ROUTINE  lpStartAddress,     //被线程执行的回调函数，也称为线程函数
    LPVOID                  lpParameter,        //传入线程函数的参数，不需传递参数时为NULL
    DWORD                   dwCreationFlags,    //控制线程创建的标志
    LPDWORD                 lpThreadId          //传出参数，用于获得线程ID，如果为NULL则不返回线程ID
);
```

**说明：**
lpThreadAttributes：指向SECURITY_ATTRIBUTES结构的指针，决定返回的句柄是否可被子进程继承，如果为NULL则表示返回的句柄不能被子进程继承。
dwStackSize ：线程栈的初始化大小，字节单位。系统分配这个值对
lpStartAddress：指向一个函数指针，该函数将被线程调用执行。因此该函数也被称为线程函数(ThreadProc)，是线程执行的起始地址，线程函数是一个回调函数，由操作系统在线程中调用。

线程函数的原型如下：
DWORD WINAPI ThreadProc(LPVOID lpParameter);    //lpParameter是传入的参数，是一个空指针
lpParameter：传入线程函数(ThreadProc)的参数，不需传递参数时为NULL
dwCreationFlags：控制线程创建的标志，有三个类型，0：线程创建后立即执行线程；CREATE\_SUSPENDED：线程创建后进入就绪状态，直到线程被唤醒时才调用；STACK\_SIZE\_PARAM\_IS\_A\_RESERVATION：dwStackSize 参数指定线程初始化栈的大小，如果STACK\_SIZE\_PARAM\_IS\_A\_RESERVATION标志未指定，dwStackSize将会设为系统预留的值。
返回值：如果线程创建成功，则返回这个新线程的句柄，否则返回NULL。如果线程创建失败，可通过GetLastError函数获得错误信息。
BOOL WINAPI CloseHandle(HANDLE hObject);        //关闭一个被打开的对象句柄
可用这个函数关闭创建的线程句柄，如果函数执行成功则返回true(非0),如果失败则返回false(0)，如果执行失败可调用GetLastError.函数获得错误信息。

创建一个简单的线程:
```cpp
#include "stdafx.h"
#include <windows.h>
#include <iostream>
using namespace std;
//线程函数
DWORD WINAPI ThreadProc(LPVOID lpParameter)
{
    for (int i = 0; i < 5; ++ i)
    {
        cout << "子线程:i = " << i << endl;
        Sleep(100);
    }
    return 0L;
}

int main()
{
    //创建一个线程
    HANDLE thread = CreateThread(NULL, 0, ThreadProc, NULL, 0, NULL);
    //关闭线程
    CloseHandle(thread);
    //主线程的执行路径
    for (int i = 0; i < 5; ++ i)
    {
        cout << "主线程:i = " << i << endl;
        Sleep(100);
    }
    return 0;
}
```

在线程函数中传入参数
```cpp
#include "stdafx.h"
#include <windows.h>
#include <iostream>
using namespace std;
#define NAME_LINE   40
//定义线程函数传入参数的结构体
typedef struct __THREAD_DATA
{
    int nMaxNum;
    char strThreadName[NAME_LINE];
    __THREAD_DATA() : nMaxNum(0)
    {
        memset(strThreadName, 0, NAME_LINE * sizeof(char));
    }
}THREAD_DATA;

//线程函数
DWORD WINAPI ThreadProc(LPVOID lpParameter)
{
    THREAD_DATA* pThreadData = (THREAD_DATA*)lpParameter;
    for (int i = 0; i < pThreadData->nMaxNum; ++ i)
    {
        cout << pThreadData->strThreadName << " --- " << i << endl;
        Sleep(100);
    }
    return 0L;
}

int main()
{
    //初始化线程数据
    THREAD_DATA threadData1, threadData2;
    threadData1.nMaxNum = 5;
    strcpy(threadData1.strThreadName, "线程1");
    threadData2.nMaxNum = 10;
    strcpy(threadData2.strThreadName, "线程2");

    //创建第一个子线程
    HANDLE hThread1 = CreateThread(NULL, 0, ThreadProc, &threadData1, 0, NULL);
    //创建第二个子线程
    HANDLE hThread2 = CreateThread(NULL, 0, ThreadProc, &threadData2, 0, NULL);
    //关闭线程
    CloseHandle(hThread1);
    CloseHandle(hThread2);

    //主线程的执行路径
    for (int i = 0; i < 5; ++ i)
    {
        cout << "主线程 === " << i << endl;
        Sleep(100);
    }

    system("pause");
    return 0;
}
```

线程同步

```cpp
#include "stdafx.h"
#include <windows.h>
#include <iostream>
#define NAME_LINE   40
//定义线程函数传入参数的结构体
typedef struct __THREAD_DATA
{
    int nMaxNum;
    char strThreadName[NAME_LINE];

    __THREAD_DATA() : nMaxNum(0)
    {
        memset(strThreadName, 0, NAME_LINE * sizeof(char));
    }
}THREAD_DATA;

HANDLE g_hMutex = NULL;     //互斥量

//线程函数
DWORD WINAPI ThreadProc(LPVOID lpParameter)
{
    THREAD_DATA* pThreadData = (THREAD_DATA*)lpParameter;

    for (int i = 0; i < pThreadData->nMaxNum; ++ i)
    {
        //请求获得一个互斥量锁
        WaitForSingleObject(g_hMutex, INFINITE);
        cout << pThreadData->strThreadName << " --- " << i << endl;
        Sleep(100);
        //释放互斥量锁
        ReleaseMutex(g_hMutex);
    }
    return 0L;
}

int main()
{
    //创建一个互斥量
    g_hMutex = CreateMutex(NULL, FALSE, NULL);

    //初始化线程数据
    THREAD_DATA threadData1, threadData2;
    threadData1.nMaxNum = 5;
    strcpy(threadData1.strThreadName, "线程1");
    threadData2.nMaxNum = 10;
    strcpy(threadData2.strThreadName, "线程2");

    //创建第一个子线程
    HANDLE hThread1 = CreateThread(NULL, 0, ThreadProc, &threadData1, 0, NULL);
    //创建第二个子线程
    HANDLE hThread2 = CreateThread(NULL, 0, ThreadProc, &threadData2, 0, NULL);
    //关闭线程
    CloseHandle(hThread1);
    CloseHandle(hThread2);

    //主线程的执行路径
    for (int i = 0; i < 5; ++ i)
    {
        //请求获得一个互斥量锁
        WaitForSingleObject(g_hMutex, INFINITE);
        cout << "主线程 === " << i << endl;
        Sleep(100);
        //释放互斥量锁
        ReleaseMutex(g_hMutex);
    }

    system("pause");
    return 0;
}
```

模拟火车售票系统
SaleTickets.h :

```cpp
#include "stdafx.h"
#include <windows.h>
#include <iostream>
#include <strstream> 
#include <string>

using namespace std;

#define NAME_LINE   40

//定义线程函数传入参数的结构体
typedef struct __TICKET
{
    int nCount;
    char strTicketName[NAME_LINE];

    __TICKET() : nCount(0)
    {
        memset(strTicketName, 0, NAME_LINE * sizeof(char));
    }
}TICKET;

typedef struct __THD_DATA
{
    TICKET* pTicket;
    char strThreadName[NAME_LINE];

    __THD_DATA() : pTicket(NULL)
    {
        memset(strThreadName, 0, NAME_LINE * sizeof(char));
    }
}THD_DATA;


 //基本类型数据转换成字符串
template<class T>
string convertToString(const T val)
{
    string s;
    std::strstream ss;
    ss << val;
    ss >> s;
    return s;
}

//售票程序
DWORD WINAPI SaleTicket(LPVOID lpParameter);
```

SaleTickets.cpp ：

```cpp
#include "stdafx.h"
#include <windows.h>
#include <iostream>
#include "SaleTickets.h"

using namespace std;

extern HANDLE g_hMutex;

//售票程序
DWORD WINAPI SaleTicket(LPVOID lpParameter)
{

    THD_DATA* pThreadData = (THD_DATA*)lpParameter;
    TICKET* pSaleData = pThreadData->pTicket;
    while(pSaleData->nCount > 0)
    {
        //请求获得一个互斥量锁
        WaitForSingleObject(g_hMutex, INFINITE);
        if (pSaleData->nCount > 0)
        {
            cout << pThreadData->strThreadName << "出售第" << pSaleData->nCount -- << "的票,";
            if (pSaleData->nCount >= 0) {
                cout << "出票成功!剩余" << pSaleData->nCount << "张票." << endl;
            } else {
                cout << "出票失败！该票已售完。" << endl;
            }
        }
        Sleep(10);
        //释放互斥量锁
        ReleaseMutex(g_hMutex);
    }

    return 0L;
}
```

测试程序：

```cpp
//售票系统
void Test2()
{
    //创建一个互斥量
    g_hMutex = CreateMutex(NULL, FALSE, NULL);

    //初始化火车票
    TICKET ticket;
    ticket.nCount = 100;
    strcpy(ticket.strTicketName, "北京-->赣州");

    const int THREAD_NUMM = 8;
    THD_DATA threadSale[THREAD_NUMM];
    HANDLE hThread[THREAD_NUMM];
    for(int i = 0; i < THREAD_NUMM; ++ i)
    {
        threadSale[i].pTicket = &ticket;
        string strThreadName = convertToString(i);

        strThreadName = "窗口" + strThreadName;

        strcpy(threadSale[i].strThreadName, strThreadName.c_str());

        //创建线程
        hThread[i] = CreateThread(NULL, NULL, SaleTicket, &threadSale[i], 0, NULL);

        //请求获得一个互斥量锁
        WaitForSingleObject(g_hMutex, INFINITE);
        cout << threadSale[i].strThreadName << "开始出售 " << threadSale[i].pTicket->strTicketName << " 的票..." << endl;
        //释放互斥量锁
        ReleaseMutex(g_hMutex);

        //关闭线程
        CloseHandle(hThread[i]);
    }

    system("pause");
}
```

### 进程的创建


### 进程间同步方式
1. Mutex（互斥）可以跨进城使用
2. Semphore（信号量）可以跨进城使用等


### 进程间的通信方式
进程间通信又称IPC(Inter-Process Communication),指多个进程之间相互通信，交换信息的方法。
进程间通讯(IPC)方法主要有以下几种: 管道/FIFO/共享内存/消息队列/信号 
根据进程通信时信息量大小的不同,可以将进程通信划分为两大类型:
1、低级通信,控制信息的通信(主要用于进程之间的同步,互斥,终止和挂起等等控制信息的传递)
2、高级通信,大批数据信息的通信(主要用于进程间数据块数据的交换和共享,常见的高级通信有管道,消息队列,共享内存等). 

1.管道有命名管道和非命名管道(即匿名管道)之分，非命名管道(即匿名管道)只能用于父子进程通讯，命名管道可用于非父子进程，命名管道就是FIFO，管道是先进先出的通讯方式.  
管道( pipe )：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
命名管道 (named pipe) ： 命名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。

2.消息队列是用于两个进程之间的通讯，首先在一个进程中创建一个消息队列，然后再往消息队列中写数据，而另一个进程则从那个消息队列中取数据。
需要注意的是，消息队列是用创建文件的方式建立的，如果一个进程向某个消息队列中写入了数据之后，另一个进程并没有取出数据，即使向消息队列中写数据的进程已经结束，保存在消息队列中的数据并没有消失，也就是说下次再从这个消息队列读数据的时候，就是上次的数据！! 
消息队列( message queue ) ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。
消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。  

3.信号量，它与WINDOWS下的信号量是一样的.
信号量( semophore ) ： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。不是用于交换大批数据,而用于多线程之间的同步.常作为一种锁机制,防止某进程在访问资源时其它进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。

4.共享内存，类似于WINDOWS下的DLL中的共享变量，但LINUX下的共享内存区不需要像DLL这样的东西，只要首先创建一个共享内存区，其它进程按照一定的步骤就能访问到这个共享内存区中的数据，当然可读可写.
共享内存( shared memory )：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。
共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量，配合使用，来实现进程间的同步和通信。    
信号 ( signal ) ： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。

套接字( socket ) ：套接字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同机器间的进程通信

#### 参考
* [编程思想之多线程与多进程](http://blog.csdn.net/luoweifu/article/details/46595285)
* [知乎](https://www.zhihu.com/question/36529093/answer/67959045)