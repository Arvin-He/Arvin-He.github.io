---
title: Python之进程
date: 2017-06-12 16:17:27
tags: Python
categories: 编程
---

### 多进程模块
由于GIL（全局解释锁）的问题，多线程并不能充分利用多核处理器，如果是一个CPU计算型的任务，应该使用多进程模块 multiprocessing 。它的工作方式与线程库完全不同，但是两种库的语法却非常相似。multiprocessing给每个进程赋予单独的Python解释器，这样就规避了全局解释锁所带来的问题。下面将讲述多进程之间的通信
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
print parent_conn.recv()
p.join()
```

其中Pipe返回的是管道2边的对象：「父连接」和「子连接」。当子连接发送一个带有hello字符串的列表，父连接就会收到，所以parent_conn.recv()就会打印出来。这样就可以简单的实现在多进程之间传输Python内置的数据结构了。但是先说明，不能被xmlrpclib序列化的对象是不能这么传输的。




### 参考
* [理解Python并发编程 - 进程篇](https://juejin.im/post/5847853661ff4b006c431c64)