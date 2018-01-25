---
title: cookbook之并发编程
date: 2017-07-05 10:44:40
tags: python
categories: python
---
### 启动与停止线程
```python
# Code to execute in an independent thread
import time

def countdown(n):
    while n > 0:
        print('T-minus', n)
        n -= 1
        time.sleep(5)

# Create and launch a thread
from threading import Thread
t = Thread(target=countdown, args=(10,))
t.start()
```

当你创建好一个线程对象后，该对象并不会立即执行，除非你调用它的 start()方法（当你调用 start() 方法时，它会调用你传递进来的函数，并把你传递进来的参数传递给该函数）。Python 中的线程会在一个单独的系统级线程中执行（比如说一个POSIX 线程或者一个 Windows 线程），这些线程将由操作系统来全权管理。线程一旦启动，将独立执行直到目标函数返回。你可以查询一个线程对象的状态，看它是否还在执行：
```python
if t.is_alive():
    print('Still running')
else:
    print('Completed')
```

你也可以将一个线程加入到当前线程，并等待它终止：`t.join()`
Python 解释器在所有线程都终止后才继续执行代码剩余的部分。对于需要长时间运行的线程或者需要一直运行的后台任务，你应当考虑使用后台线程。
```python
# 后台线程, 指定daemon为true
t = Thread(target=countdown, args=(10,), daemon=True)
t.start()
```

后台线程无法等待，不过，这些线程会在主线程终止时自动销毁。

###  判断线程是否已经启动
使用 threading 库中的 Event 对象。Event 对象包含一个可由线程设置的信号标志，它允许线程等待某些事件的发生。在初始情况下，event 对象中的信号标志被设置为假。如果有线程等待一个 event 对象，而这个 event 对象的标志为假，那么这个线程将会被一直阻塞直至该标志为真。一个线程如果将一个 event 对象的信号标志设置为真，它将唤醒所有等待这个 event 对象的线程。如果一个线程等待一个已经被设置为真的 event 对象，那么它将忽略这个事件，继续执行。

event 对象最好单次使用，就是说，你创建一个 event 对象，让某个线程等待这个对象，一旦这个对象被设置为真，你就应该丢弃它。尽管可以通过 clear() 方法来重置 event 对象，但是很难确保安全地清理 event 对象并对它重新赋值。很可能会发生错过事件、死锁或者其他问题（特别是，你无法保证重置 event 对象的代码会在线程再次等待这个 event 对象之前执行）。如果一个线程需要不停地重复使用 event 对象，你最好使用 Condition 对象来代替。

event 对象的一个重要特点是当它被设置为真时会唤醒所有等待它的线程。如果你只想唤醒单个线程，最好是使用信号量或者 Condition 对象来替代。

编写涉及到大量的线程间同步问题的代码会让你痛不欲生。比较合适的方式是使用队列来进行线程间通信或者每个把线程当作一个 Actor，利用 Actor 模型来控制并发。

###  线程间通信
从一个线程向另一个线程发送数据最安全的方式可能就是使用 queue 库中的队列了。创建一个被多个线程共享的 Queue 对象，这些线程通过使用 put() 和 get() 操作来向队列中添加或者删除元素。
Queue 对象已经包含了必要的锁，所以你可以通过它在多个线程间多安全地共享数据。当使用队列时，协调生产者和消费者的关闭问题可能会有一些麻烦。一个通用的解决方法是在队列中放置一个特殊的值，当消费者读到这个值的时候，终止执行。
```python
from queue import Queue
from threading import Thread
# Object that signals shutdown
_sentinel = object()
# A thread that produces data
def producer(out_q):
    while running:
        # Produce some data
        ...
        out_q.put(data)
    # Put the sentinel on the queue to indicate completion
    out_q.put(_sentinel)

# A thread that consumes data
def consumer(in_q):
    while True:
        # Get some data
        data = in_q.get()
        # Check for termination
        if data is _sentinel:
            in_q.put(_sentinel)
            break
        # Process the data
        ...
```

本例中有一个特殊的地方：消费者在读到这个特殊值之后立即又把它放回到队列中，将之传递下去。这样，所有监听这个队列的消费者线程就可以全部关闭了。

创建一个线程安全的优先级队列
```python
import heapq
import threading

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._count = 0
        self._cv = threading.Condition()

    def put(self, item, priority):
        with self._cv:
            heapq.heappush(self._queue, (-priority, self._count, item))
            self._count += 1
            self._cv.notify()
    
    def get(self):
        with self._cv:
            while len(self._queue) == 0:
                self._cv.wait()
            return heapq.heappop(self._queue)[-1]
```

使用队列来进行线程间通信是一个单向、不确定的过程。通常情况下，你没有办法知道接收数据的线程是什么时候接收到的数据并开始工作的。
使用线程队列有一个要注意的问题是，向队列中添加数据项时并不会复制此数据项，线程间通信实际上是在线程间传递对象引用。如果你担心对象的共享
状态，那你最好只传递不可修改的数据结构（如：整型、字符串或者元组）或者一个对象的深拷贝。

### 给关键部分加锁
使用 threading 库中的 Lock 对象
```python
import threading

class SharedCounter:
'''
A counter object that can be shared by multiple threads.
'''
def __init__(self, initial_value = 0):
    self._value = initial_value
    self._value_lock = threading.Lock()

def incr(self,delta=1):
'''
Increment the counter with locking
'''
    with self._value_lock:
        self._value += delta

def decr(self,delta=1):
'''
Decrement the counter with locking
'''
    with self._value_lock:
        self._value -= delta
```

Lock 对象和 with 语句块一起使用可以保证互斥执行，就是每次只有一个线程可以执行 with 语句包含的代码块。with 语句会在这个代码块执行前自动获取锁，在执行结束后自动释放锁。

线程调度本质上是不确定的，因此，在多线程程序中错误地使用锁机制可能会导致随机数据损坏或者其他的异常行为，我们称之为竞争条件。为了避免竞争条件，最好只在临界区（对临界资源进行操作的那部分代码）使用锁。

关于信号量
信号量对象是一个建立在共享计数器基础上的同步原语。如果计数器不为 0，with 语句将计数器减 1，线程被允许执行。with 语句执行结束后，计数器加１。如果计数器为 0，线程将被阻塞，直到其他线程结束将计数器加 1。尽管你可以在程序中像标准锁一样使用信号量来做线程同步，但是这种方式并不被推荐，因为使用信号量为程序增加的复杂性会影响程序性能。相对于简单地作为锁使用，信号量更适用于那些需要在线程之间引入信号或者限制的程序。比如，你需要限制一段代码的并发访问量，你就可以像下面这样使用信号量完成：
```python
from threading import Semaphore
import urllib.request

# At most, five threads allowed to run at once
_fetch_url_sema = Semaphore(5)

def fetch_url(url):
    with _fetch_url_sema:
        return urllib.request.urlopen(url)
```

### 防止死锁的加锁机制
在多线程程序中，死锁问题很大一部分是由于线程同时获取多个锁造成的.举个例子：一个线程获取了第一个锁，然后在获取第二个锁的时候发生阻塞，那么这个线程就可能阻塞其他线程的执行，从而导致整个程序假死。解决死锁问题的一种方案是为程序中的每一个锁分配一个唯一的 id，然后只允许按照升序规则来使用多个锁，这个规则使用上下文管理器是非常容易实现的，示例如下：
```python
import threading
from contextlib import contextmanager

# Thread-local state to stored information on locks already acquired
_local = threading.local()

@contextmanager
def acquire(*locks):
    # Sort locks by object identifier
    locks = sorted(locks, key=lambda x: id(x))

    # Make sure lock order of previously acquired locks is not violated
    acquired = getattr(_local,'acquired',[])
    if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
        raise RuntimeError('Lock Order Violation')

    # Acquire all of the locks
    acquired.extend(locks)
    _local.acquired = acquired

    try:
        for lock in locks:
            lock.acquire()
        yield
    finally:
        # Release locks in reverse order of acquisition
        for lock in reversed(locks):
            lock.release()
        del acquired[-len(locks):]

```

```python
import threading

x_lock = threading.Lock()
y_lock = threading.Lock()

def thread_1():
    while True:
        with acquire(x_lock, y_lock):
            print('Thread-1')

def thread_2():
    while True:
        with acquire(y_lock, x_lock):
            print('Thread-2')

t1 = threading.Thread(target=thread_1)
t1.daemon = True
t1.start()

t2 = threading.Thread(target=thread_2)
t2.daemon = True
t2.start()

# 下面的写法会导致线程崩溃,发生崩溃的原因在于，每个线程都记录着自己已经获取到的锁。 
# acquire() 函数会检查之前已经获取的锁列表，由于锁是按照升序排列获取的，
# 所以函数会认为之前已获取的锁的 id 必定小于新申请到的锁，这时就会触发异常。
def thread_1():
    while True:
        with acquire(x_lock):
            with acquire(y_lock):
                print('Thread-1')

def thread_2():
    while True:
        with acquire(y_lock):
            with acquire(x_lock):
                print('Thread-2')
```

死锁是每一个多线程程序都会面临的一个问题,根据经验来讲，尽可能保证每一个线程只能同时保持一个锁，这样程序
就不会被死锁问题所困扰。一旦有线程同时申请多个锁，一切就不可预料了。

死锁的检测与恢复是一个几乎没有优雅的解决方案的扩展话题。一个比较常用的死锁检测与恢复的方案是引入看门狗计数器。当线程正常运行的时候会每隔一段时间重置计数器，在没有发生死锁的情况下，一切都正常进行。一旦发生死锁，由于无法重置计数器导致定时器超时，这时程序会通过重启自身恢复到正常状态。

避免死锁是另外一种解决死锁问题的方式，在进程获取锁的时候会严格按照对象 id升序排列获取，经过数学证明，这样保证程序不会进入死锁状态。避免死锁的主要思想是，单纯地按照对象 id 递增的顺序加锁不会产生循环依赖，而循环依赖是死锁的一个必要条件，从而避免程序进入死锁状态。

要特别注意到，为了避免死锁，所有的加锁操作必须使用 acquire() 函数。如果代码中的某部分绕过 acquire 函数直接申请锁，那么整个死锁避免机制就不起作用了。

### 创建一个线程池
通常，你应该只在 I/O 处理相关代码中使用线程池,创建大的线程池的一个可能需要关注的问题是内存的使用.

### 实现消息发布/订阅模型
要实现发布/订阅的消息通信模式，你通常要引入一个单独的“交换机”或“网关”对象作为所有消息的中介。也就是说，不直接将消息从一个任务发送到另一个，而是将其发送给交换机，然后由交换机将它发送给一个或多个被关联任务。