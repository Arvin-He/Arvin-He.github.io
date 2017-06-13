---
title: Python之进程
date: 2017-06-12 16:17:27
tags: Python
categories: 编程
---

### 多进程模块(multiprocessing)
multiprocessing是多进程模块，多进程提供了任务并发性，能充分利用多核处理器, 避免了GIL（全局解释锁）对资源的影响。由于GIL（全局解释锁）的问题，多线程并不能充分利用多核处理器，如果是一个CPU计算型的任务，应该使用多进程模块 multiprocessing 。它的工作方式与线程库完全不同，但是两种库的语法却非常相似。multiprocessing给每个进程赋予单独的Python解释器，这样就规避了全局解释锁所带来的问题。

#### multiprocessing常用类:
1. `Process(group=None, target=None, name=None, args=(), kwargs={})`
派生一个进程对象，然后调用start()方法启动
2. `Pool(processes=None, initializer=None, initargs=())`
返回一个进程池对象，processes进程池进程数量
3. `Pipe(duplex=True)`
返回两个连接对象由管道连接
4. `Queue(maxsize=0)`
返回队列对象，操作方法跟Queue.Queue一样
5. `multiprocessing.dummy`
这个库是用于实现多线程


#### Process()类有以下些方法：
`run()`
`start()` :启动进程对象
`join([timeout])` :等待子进程终止，才返回结果。可选超时。
`name` : 进程名字
`is_alive()` :返回进程是否存活
`daemon` : 进程的守护标记，一个布尔值
`pid`: 返回进程ID
`exitcode` :子进程退出状态码
`terminate()` :终止进程。在unix上使用SIGTERM信号，在windows上使用TerminateProcess()。

#### Pool()类有以下些方法：
`apply(func, args=(), kwds={})` :等效内建函数apply()
`apply_async(func, args=(), kwds={}, callback=None)` : 异步，等效内建函数apply()
`map(func, iterable, chunksize=None)` : 等效内建函数map()
`map_async(func, iterable, chunksize=None, callback=None)` :异步，等效内建函数map()
`imap(func, iterable, chunksize=1)` :等效内建函数itertools.imap()
`imap_unordered(func, iterable, chunksize=1)` :像imap()方法，但结果顺序是任意的
`close()` :关闭进程池
`terminate()` : 终止工作进程，垃圾收集连接池对象
`join()` :等待工作进程退出。必须先调用close()或terminate()

#### `Pool.apply_async()`和`Pool.map_aysnc()`又提供了以下几个方法：
`get([timeout])` : 获取结果对象里的结果。如果超时没有，则抛出TimeoutError异常
`wait([timeout])` : 等待可用的结果或超时
`ready()` : 返回调用是否已经完成
`successful()`




下面将讲述多进程之间的通信
```python
import time
import multiprocessing


def profile(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        func(*args, **kwargs)
        end   = time.time()
        print('COST: {}'.format(end - start))
    return wrapper


def fib(n):
    if n<= 2:
        return 1
    return fib(n-1) + fib(n-2)


@profile
def nomultiprocess():
    fib(35)
    fib(35)


@profile
def hasmultiprocess():
    jobs = []
    for i in range(2):
        # 注意这里传参数,会报错,需要在执行前加上 freeze_support()
        p = multiprocessing.Process(target=fib, args=(35,))
        p.start()
        jobs.append(p)

    for p in jobs:
        p.join()

if __name__ == "__main__":
    multiprocessing.freeze_support()
    nomultiprocess()
    hasmultiprocess()
```

运行结果:
```
COST: 5.3473060131073
COST: 3.7542150020599365
```

### 进程池
**有一点要强调：**任务的执行周期决定于CPU核数和任务分配算法。
```python
from multiprocessing import Pool

pool = Pool(2)
pool.map(fib, [35] * 2)
```

上面的hasmultiprocess函数的用法非常中规中矩且常见，但是我认为更好的写法是使用Pool，也就是对应线程池的进程池. 其中map方法用起来和内置的map函数一样，却有多进程的支持。

### dummy模块
当出现要从多线程改成多进程或者多进程改成多线程的时候，可以使用multiprocessing.dummy这个子模块，dummy」这个词有「模仿」的意思，如果分不清任务是CPU密集型还是IO密集型就是用这个模块. `from multiprocessing.dummy import Pool`, 这样在多线程/多进程之间切换非常方便。如果一个任务拿不准是CPU密集还是I/O密集型，且没有其它不能选择多进程方式的因素，都统一直接上多进程模式。

### 基于Pipe的parmap
进程间的通信（IPC）常用的是rpc、socket、pipe（管道）和消息队列（queue）。
多进程模块中涉及到了后面3种。先看一个官网给出的最基本的管道的例子：
```python
from multiprocessing import Process, Pipe

def f(conn):
    conn.send(['hello'])
    conn.close()

parent_conn, child_conn = Pipe()
p = Process(target=f, args=(child_conn,))
p.start()
print(parent_conn.recv())
p.join()
```

其中Pipe返回的是管道2边的对象：「父连接」和「子连接」。当子连接发送一个带有hello字符串的列表，父连接就会收到，所以parent_conn.recv()就会打印出来。这样就可以简单的实现在多进程之间传输Python内置的数据结构了。但是先说明，不能被xmlrpclib序列化的对象是不能这么传输的。

### 队列

### 同步机制
multiprocessing的Lock、Condition、Event、RLock、Semaphore等同步原语和threading模块的机制是一样的，用法也类似.

### 进程间共享状态
multiprocessing提供的在进程间共享状态的方式有2种：

#### 共享内存
主要通过Value或者Array来实现。常见的共享的有以下几种：
```python
In [1]: from multiprocessing.sharedctypes import typecode_to_type

In [2]: typecode_to_type
Out[2]:
{'B': ctypes.c_ubyte,
 'H': ctypes.c_ushort,
 'I': ctypes.c_ulong,
 'L': ctypes.c_ulong,
 'b': ctypes.c_byte,
 'c': ctypes.c_char,
 'd': ctypes.c_double,
 'f': ctypes.c_float,
 'h': ctypes.c_short,
 'i': ctypes.c_long,
 'l': ctypes.c_long,
 'u': ctypes.c_wchar}
 ```

 而且共享的时候还可以给Value或者Array传递lock参数来决定是否带锁，如果不指定默认为RLock。

 ### 进程间对象共享
一个multiprocessing.Manager对象会控制一个服务器进程，其他进程可以通过代理的方式来访问这个服务器进程。
常见的共享方式有以下几种：
1. Namespace。创建一个可分享的命名空间。
2. Value/Array。和上面共享ctypes对象的方式一样。
3. dict/list。创建一个可分享的dict/list，支持对应数据结构的方法。
4. Condition/Event/Lock/Queue/Semaphore。创建一个可分享的对应同步原语的对象。

### 分布式的进程间通信
用Manager和Queue就可以实现简单的分布式的不同服务器的不同进程间的通信（C/S模式）。

### 参考
* [理解Python并发编程 - 进程篇](https://juejin.im/post/5847853661ff4b006c431c64)
* [Python多进程与多线程](https://yq.aliyun.com/articles/65091?utm_campaign=wenzhang&utm_medium=article&utm_source=QQ-qun&utm_content=m_8078)