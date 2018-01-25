---
title: Python之线程
date: 2017-06-12 10:25:11
tags: python
categories: python
---
### 关于GIL
Python（特指CPython）的多线程不能利用多核的优势，这是因为全局解释锁（GIL）的限制。如果是cpu密集型(计算型)的任务，使用多线程GIL就会让多线程变慢。
GIL是必须的，这是Python设计的问题：Python解释器是非线程安全的。这意味着当从线程内尝试安全的访问Python对象的时候将有一个全局的强制锁。 在任何时候，仅仅一个单一的线程能够获取Python对象或者C API。每100个字节的Python指令解释器将重新获取锁，这（潜在的）阻塞了I/O操作。因为锁，CPU密集型的代码使用线程库时，不会获得性能的提高（但是当它使用之后介绍的多进程库时，性能可以获得提高）。

### 线程同步机制
Python线程包含多种同步机制:
1. Semaphore（信号量）
2. Lock（锁）
3. RLock（可重入锁）
4. Condition（条件）
5. Event
6. Queue

### Semaphore（信号量）
在多线程编程中，为了防止不同的线程同时对一个公用的资源（比如全局变量）进行修改，需要进行同时访问的数量（通常是1）。
信号量同步基于内部计数器，每调用一次acquire()，计数器减1；每调用一次release()，计数器加1.当计数器为0时，acquire()调用被阻塞。
```python
import time
from random import random
from threading import Thread, Semaphore

sema = Semaphore(3)


def foo(tid):
    with sema:
        print('{} acquire sema'.format(tid))
        wt = random() * 2
        time.sleep(wt)
    print('{} release sema'.format(tid))


threads = []

for i in range(5):
    t = Thread(target=foo, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

这个例子中，限制了同时能访问资源的数量为3。看一下运行的效果：
```
0 acquire sema
1 acquire sema
2 acquire sema
0 release sema
3 acquire sema
2 release sema
4 acquire sema
1 release sema
3 release sema
4 release sema
```

### Lock（锁）
Lock也可以叫做**互斥锁**，其实相当于信号量为1。
不加锁：
```python
import time
from threading import Thread

value = 0


def foo():
    global value
    newvalue = value + 1
    time.sleep(0.001)  # 使用sleep让线程有机会切换
    value = newvalue


threads = []

for i in range(100):
    t = Thread(target=foo)
    t.start()
    threads.append(t)

# 创建了100个线程
print(len(threads))

for t in threads:
    t.join()

print(value)
```

运行结果:
```
100
10
```

加锁
```python
import time
from threading import Thread, Lock

# 全局变量
value = 0
lock = Lock()


def foo():
    global value
    # 加锁,执行完会自动释放锁
    with lock:
        new = value + 1
        time.sleep(0.001)
        value = new

threads = []

for i in range(100):
    t = Thread(target=foo)
    t.start()
    threads.append(t)

for t in threads:
    t.join()

print(value)
```

运行结果:100

### RLock（可重入锁）
acquire() 能够不被阻塞的被同一个线程调用多次。但是要注意的是release()需要调用与acquire()相同的次数才能释放锁。

### Condition（条件）
一个线程等待特定条件，而另一个线程发出特定条件满足的信号。最好说明的例子就是「生产者/消费者」模型：
```python
import time
import threading


def consumer(cond):
    t = threading.currentThread()
    with cond:
        # wait()方法创建了一个名为waiter的锁，
        # 并且设置锁的状态为locked。这个waiter锁用于线程间的通讯
        cond.wait()
        print('{}: Resource is available to consumer'.format(t.name))


def producer(cond):
    t = threading.currentThread()
    with cond:
        print('{}: Making resource available'.format(t.name))
        # 释放waiter锁，唤醒消费者
        cond.notifyAll()


condition = threading.Condition()

c1 = threading.Thread(name='c1', target=consumer, args=(condition,))
c2 = threading.Thread(name='c2', target=consumer, args=(condition,))
p = threading.Thread(name='p', target=producer, args=(condition,))

c1.start()
time.sleep(1)
c2.start()
time.sleep(1)
p.start()
```

运行结果:
```
p: Making resource available
c2: Resource is available to consumer
c1: Resource is available to consumer
```

可以看到生产者发送通知之后，消费者都收到了。

### Event
一个线程发送/传递事件，另外的线程等待事件的触发.同样的用「生产者/消费者」模型的例子:

```python
import time
import threading
from random import randint

TIMEOUT = 2


def consumer(event, mylist):
    t = threading.currentThread()
    while 1:
        event_is_set = event.wait(TIMEOUT)
        if event_is_set:
            try:
                integer = mylist.pop()
                print('{} popped from list by {}'.format(integer, t.name))
                # 重置事件状态
                event.clear()
            except IndexError:
                # 为了让刚启动时容错
                pass


def producer(event, mylist):
    t = threading.currentThread()
    while 1:
        integer = randint(10, 100)
        mylist.append(integer)
        print('{} appended to list by {}'.format(integer, t.name))
        # 设置事件
        event.set()
        time.sleep(1)


event = threading.Event()
mylist = []
threads = []

for name in ('consumer1', 'consumer2'):
    t = threading.Thread(name=name, target=consumer, args=(event, mylist))
    t.start()
    threads.append(t)

p = threading.Thread(name='producer1', target=producer, args=(event, mylist))
p.start()
threads.append(p)

for t in threads:
    t.join()

```

运行结果:
```
86 appended to list by producer1 
86 popped from list by consumer1 
29 appended to list by producer1 
29 popped from list by consumer1 
36 appended to list by producer1 
36 popped from list by consumer2 
47 appended to list by producer1 
47 popped from list by consumer2 
16 appended to list by producer1 
16 popped from list by consumer1 
95 appended to list by producer1 
95 popped from list by consumer1 
51 appended to list by producer1 
51 popped from list by consumer1 
36 appended to list by producer1 
36 popped from list by consumer1 
12 appended to list by producer1 
12 popped from list by consumer1 
12 appended to list by producer1 
12 popped from list by consumer1 
```

可以看到事件被2个消费者比较平均的接收并处理了。如果使用了wait方法，线程就会等待我们设置事件，这也有助于保证任务的完成。

### Queue
队列在并发开发中最常用的。我们借助「生产者/消费者」模式来理解：
生产者把生产的「消息」放入队列，消费者从这个队列中对去对应的消息执行。

大家主要关心如下4个方法就好了：

1. put: 向队列中添加一个项。
2. get: 从队列中删除并返回一个项。
3. task_done: 当某一项任务完成时调用。
4. join: 阻塞直到所有的项目都被处理完。

```python
import time
import threading
from random import random
from queue import Queue

q = Queue()


def double(n):
    return n * 2


def producer():
    while 1:
        wt = random()
        time.sleep(wt)
        q.put((double, wt))


def consumer():
    while 1:
        task, arg = q.get()
        print(arg, task(arg))
        q.task_done()

for target in(producer, consumer):
    t = threading.Thread(target=target)
    t.start()
```

运行结果:
```
0.5001101134617869 1.0002202269235738
0.2397443354990395 0.479488670998079
0.018426830503480485 0.03685366100696097
0.9260989761246562 1.8521979522493124
0.808116115591099 1.616232231182198
0.5868108877921562 1.1736217755843124
0.5195607837070528 1.0391215674141057
0.32311190835552184 0.6462238167110437
```

这就是最简化的队列架构。

Queue模块还自带了PriorityQueue（带有优先级）和LifoQueue（后进先出）2种特殊队列。
下面展示线程安全的优先级队列的用法，
PriorityQueue要求我们put的数据的格式是(priority_number, data)，我们看看下面的例子：

```python
import time
import threading
from random import randint
from queue import PriorityQueue

q = PriorityQueue()


def double(n):
    return n*2


def producer():
    count = 0
    while 1:
        if count > 5:
            break
        pri = randint(0, 100)
        print('put :{}'.format(pri))
        # (priority, func, args)
        q.put((pri, double, pri))
        count += 1


def consumer():
    while 1:
        if q.empty():
            break
        pri, task, arg = q.get()
        print('[PRI:{}] {} * 2 = {}'.format(pri, arg, task(arg)))
        q.task_done()
        time.sleep(0.1)


t = threading.Thread(target=producer)
t.start()
time.sleep(1)
t = threading.Thread(target=consumer)
t.start()
```

运行结果:
```
put :54
put :70
put :62
put :54
put :20
put :75
[PRI:20] 20 * 2 = 40
[PRI:54] 54 * 2 = 108
[PRI:54] 54 * 2 = 108
[PRI:62] 62 * 2 = 124
[PRI:70] 70 * 2 = 140
[PRI:75] 75 * 2 = 150
```

可以看到put时的数字是随机的，但是get时先从优先级更高（数字小表示优先级高）开始获取的。

### 线程池
面向对象开发中，创建和销毁对象是很费时间的，因为创建一个对象要获取内存资源或者其它更多资源。无节制的创建和销毁线程是一种极大的浪费。那我们可不可以把执行完任务的线程不销毁而重复利用呢？仿佛就是把这些线程放进一个池子，一方面我们可以控制同时工作的线程数量，一方面也避免了创建和销毁产生的开销。

线程池在标准库中其实是有体现的，只是在官方文章中基本没有被提及：
```python
In [1]: from multiprocessing.pool import ThreadPool

In [2]: pool = ThreadPool(5)

In [3]: pool.map(lambda x: x**2, range(5))
Out[3]: [0, 1, 4, 9, 16]
```

自己实现:
```python
import time
import threading
from random import random
from queue import Queue


def double(n):
    return n*2


class Worker(threading.Thread):
    def __init__(self, queue):
        super(Worker, self).__init__()
        self._q = queue
        self.daemon = True
        self.start()

    def run(self):
        while 1:
            f, args, kwargs = self._q.get()
            try:
                # 线程名字
                print('USE: {}'.format(self.name))
                print(f(*args, **kwargs))
            except Exception as e:
                print(e)
            self._q.task_done()


class ThreadPool(object):
    def __init__(self, num_t=5):
        self._q = Queue(num_t)
        # 创建5个工作线程
        for _ in range(num_t):
            Worker(self._q)

    def add_task(self,  f,  *args, **kwargs):
        self._q.put((f, args, kwargs))

    def wait_complete(self):
        self._q.join()


pool = ThreadPool()
for _ in range(8):
    wt = random()
    pool.add_task(double, wt)
    time.sleep(wt)

pool.wait_complete()
```

运行结果:
```
USE: Thread-1
0.4563649806005714
USE: Thread-2
1.818831738188373
USE: Thread-3
1.3641601633838014
USE: Thread-4
1.4812490759517853
USE: Thread-5
0.9838021089438205
USE: Thread-1
0.5131452235979674
USE: Thread-2
1.7305538822346334
USE: Thread-3
1.8682661663096352
```

线程池会保证同时提供5个线程工作，但是我们有8个待完成的任务，可以看到线程按顺序被循环利用了。


### 参考
* [理解Python并发编程 - 线程篇](https://juejin.im/post/5845134da22b9d006c2959c3)
