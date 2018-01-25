---
title: Python之进程
date: 2017-06-12 16:17:27
tags: python
categories: python
---

### 多进程模块(multiprocessing)简介
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

### multiprocessing使用
一个简单的例子
```python
from multiprocessing import Process
import os

def worker(name):
    print(name)
    print("parent pid = {}".format(os.getppid()))
    print("current pid = {}".format(os.getpid()))

if __name__ == "__main__":
    p = Process(target=worker, args=('func worker', ))
    p.start()
    p.join()
    print(p.name)

#输出结果
func worker
parent pid = 5476
current pid = 5992
Process-1
```

关于`join([timeout])`方法说明:如果可选参数timeout为None(默认值)，该方法将阻塞，直到调用join()方法的进程终止。如果超时为正数，则阻塞最多超时timeout的设定值。请注意，如果方法终止或方法超时，该方法返回None。检查进程的exitcode以确定是否终止。

给子进程命名,方便管理
```python
from multiprocessing import Process
import os


def worker1():
    print("this is worker1 func")
    print("current pid = {}".format(os.getpid()))


def worker2():
    print("this is worker2 func")
    print("current pid = {}".format(os.getpid()))


if __name__ == "__main__":
    print("parent pid = {}".format(os.getppid()))
    for n in range(3):
        p1 = Process(name="worker1", target=worker1)
        p1.start()
        p1.join()
        print("child process name = {}".format(p1.name))
    p2 = Process(name="worker2", target=worker2)
    p2.start()
    p2.join()
    print("child process name = {}".format(p2.name))

# 输出结果
parent pid = 2816
this is worker1 func
current pid = 2428
child process name = worker1
this is worker1 func
current pid = 3192
child process name = worker1
this is worker1 func
current pid = 4736
child process name = worker1
this is worker2 func
current pid = 976
child process name = worker2
```

关于daemon
这里的daemon不同于linux中守护进程的概念,这里的daemon参数是一个布尔值,
如果daemon为None则创建子进程时daemon参数从父进程继承过来.
如果daemon为true,则创建的子进程随着父进程退出而退出,
如果daemon为false,则创建的子进程随着父进程退出而不退出,
注意: 一个守护进程不允许创建子进程,否则当父进程退出后,该守护进程终止后会使由该守守护进守护进程创建的子进程变成独立进程,此外，它们不是Unix守护程序或服务，它们是正常进程，如果非守护进程已退出，则该进程将被终止（并且未加入）。


### 进程池
**有一点要强调：**任务的执行周期决定于CPU核数和任务分配算法。
```python
from multiprocessing import Pool, current_process
import os, time, sys

def worker(n):
    print('hello world', n)
    # 获取当前进程名字
    print('process name:', current_process().name)
    # 休眠用于执行时有时间查看当前执行的进程
    time.sleep(1)

if __name__ == '__main__':
    p = Pool(processes=3)
    for i in range(8):
        r = p.apply_async(worker, args=(i,))
        # 获取结果中的数据
        r.get(timeout=5)  
    p.close()

# 运行结果:
hello world 0
process name: SpawnPoolWorker-2
hello world 1
process name: SpawnPoolWorker-3
hello world 2
process name: SpawnPoolWorker-1
hello world 3
process name: SpawnPoolWorker-2
hello world 4
process name: SpawnPoolWorker-3
hello world 5
process name: SpawnPoolWorker-1
hello world 6
process name: SpawnPoolWorker-2
hello world 7
process name: SpawnPoolWorker-3
```

进程池生成了3个子进程，通过循环执行8次worker函数，进程池会从子进程1开始去处理任务，当到达最大进程时，会继续从子进程1开始。

进程池map方法, map()方法是将序列中的元素通过函数处理返回新列表。
```python
from multiprocessing import Pool

def worker(url):
    return 'http://%s' % url
urls = ['www.baidu.com', 'www.jd.com']
pool = Pool(2)s
r = pool.map(worker, urls)
pool.close()
print(r)
```

上面的hasmultiprocess函数的用法非常中规中矩且常见，但是我认为更好的写法是使用Pool，也就是对应线程池的进程池. 其中map方法用起来和内置的map函数一样，却有多进程的支持。

### dummy模块
当出现要从多线程改成多进程或者多进程改成多线程的时候，可以使用multiprocessing.dummy这个子模块，dummy」这个词有「模仿」的意思，如果分不清任务是CPU密集型还是IO密集型就是用这个模块. `from multiprocessing.dummy import Pool`, 这样在多线程/多进程之间切换非常方便。如果一个任务拿不准是CPU密集还是I/O密集型，且没有其它不能选择多进程方式的因素，都统一直接上多进程模式。

### 基于Pipe的parmap
进程间的通信（IPC）常用的是rpc、socket、pipe（管道）和消息队列（queue）。
ultiprocessing支持两种类型进程间通信：Queue和Pipe。
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