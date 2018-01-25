---
title: Python之gevent使用
date: 2017-06-06 13:08:03
tags: python
categories: python
---
### Greenlets
在gevent中用到的主要模式是Greenlet, 它是以C扩展模块形式接入Python的轻量级协程。 Greenlet全部运行在主程序操作系统进程的内部，但它们被协作式地调度。**在任何时刻，只有一个协程在运行。** 在gevent里面，上下文切换是通过yielding来完成的. 当我们在受限于网络或IO的函数中使用gevent，这些函数会被协作式的调度， gevent的真正能力会得到发挥。Gevent处理了所有的细节， 来保证你的网络库会在可能的时候，隐式交出greenlet上下文的执行权。 
greenlet具有确定性。在相同配置相同输入的情况下，它们总是 会产生相同的输出。
即使gevent通常带有确定性，当开始与如socket或文件等外部服务交互时， 不确定性也可能溜进你的程序中。因此尽管gevent线程是一种“确定的并发”形式， 使用它仍然可能会遇到像使用POSIX线程或进程时遇到的那些问题。涉及并发长期存在的问题就是竞争条件(race condition)。简单来说， 当两个并发线程/进程都依赖于某个共享资源同时都尝试去修改它的时候， 就会出现竞争条件。这会导致资源修改的结果状态依赖于时间和执行顺序。 这是个问题，我们一般会做很多努力尝试避免竞争条件， 因为它会导致整个程序行为变得不确定。最好的办法是始终避免所有全局的状态。全局状态和导入时(import-time)副作用总是会 反咬你一口！

### 创建Greenlets
gevent对Greenlet初始化提供了一些封装，最常用的使用模板之一有
```python
import gevent
from gevent import Greenlet

def foo(message, n):
    """
    Each thread will be passed the message, and n arguments
    in its initialization.
    """
    gevent.sleep(n)
    print(message)

# Initialize a new Greenlet instance running the named function foo
thread1 = Greenlet.spawn(foo, "Hello", 1)

# Wrapper for creating and running a new Greenlet from the named
# function foo, with the passed arguments
thread2 = gevent.spawn(foo, "I live!", 2)

# Lambda expressions
thread3 = gevent.spawn(lambda x: (x+1), 2)

threads = [thread1, thread2, thread3]

# Block until all threads complete.
gevent.joinall(threads)


Hello
I live!
```

除使用基本的Greenlet类之外，你也可以子类化Greenlet类，重载它的_run方法。

```python

import gevent
from gevent import Greenlet

class MyGreenlet(Greenlet):

    def __init__(self, message, n):
        Greenlet.__init__(self)
        self.message = message
        self.n = n

    def _run(self):
        print(self.message)
        gevent.sleep(self.n)

g = MyGreenlet("Hi there!", 3)
g.start()
g.join()

Hi there!
```

### Greenlet状态
就像任何其他成段代码，Greenlet也可能以不同的方式运行失败。 Greenlet可能未能成功抛出异常，不能停止运行，或消耗了太多的系统资源。

一个greenlet的状态通常是一个依赖于时间的参数。在greenlet中有一些标志， 让你可以监视它的线程内部状态：

started -- Boolean, 指示此Greenlet是否已经启动
ready() -- Boolean, 指示此Greenlet是否已经停止
successful() -- Boolean, 指示此Greenlet是否已经停止而且没抛异常
value -- 任意值, 此Greenlet代码返回的值
exception -- 异常, 此Greenlet内抛出的未捕获异常

### 程序停止
当主程序(main program)收到一个SIGQUIT信号时，不能成功做yield操作的 Greenlet可能会令意外地挂起程序的执行。这导致了所谓的僵尸进程， 它需要在Python解释器之外被kill掉。对此，一个通用的处理模式就是在主程序中监听SIGQUIT信号，在程序退出 调用gevent.shutdown。
```python
import gevent
import signal

def run_forever():
    gevent.sleep(1000)

if __name__ == '__main__':
    gevent.signal(signal.SIGQUIT, gevent.shutdown)
    thread = gevent.spawn(run_forever)
    thread.join()
```

### 超时
超时是一种对一块代码或一个Greenlet的运行时间的约束。
```python
import gevent
from gevent import Timeout

seconds = 10

timeout = Timeout(seconds)
timeout.start()

def wait():
    gevent.sleep(10)

try:
    gevent.spawn(wait).join()
except Timeout:
    print('Could not complete')
```

超时类也可以用在上下文管理器(context manager)中, 也就是with语句内。
```python
import gevent
from gevent import Timeout

time_to_wait = 5 # seconds

class TooLong(Exception):
    pass

with Timeout(time_to_wait, TooLong):
    gevent.sleep(10)
```

### 猴子补丁(Monkey patching)
我们现在来到gevent的死角了. 在此之前，我已经避免提到猴子补丁(monkey patching) 以尝试使gevent这个强大的协程模型变得生动有趣，但现在到了讨论猴子补丁的黑色艺术 的时候了。你之前可能注意到我们提到了monkey.patch_socket()这个命令，这个 纯粹副作用命令是用来改变标准socket库的。
```python
import socket
print(socket.socket)

print("After monkey patch")
from gevent import monkey
monkey.patch_socket()
print(socket.socket)

import select
print(select.select)
monkey.patch_select()
print("After monkey patch")
print(select.select)

class 'socket.socket'
After monkey patch
class 'gevent.socket.socket'

built-in function select
After monkey patch
function select at 0x1924de8
```

Python的运行环境允许我们在运行时修改大部分的对象，包括模块，类甚至函数。 这是个一般说来令人惊奇的坏主意，因为它创造了“隐式的副作用”，如果出现问题 它很多时候是极难调试的。虽然如此，在极端情况下当一个库需要修改Python本身 的基础行为的时候，猴子补丁就派上用场了。在这种情况下，gevent能够 修改标准库里面大部分的阻塞式系统调用，包括socket、ssl、threading和 select等模块，而变为协作式运行。

例如，Redis的python绑定一般使用常规的tcp socket来与redis-server实例通信。 通过简单地调用gevent.monkey.patch_all()，可以使得redis的绑定协作式的调度 请求，与gevent栈的其它部分一起工作。

这让我们可以将一般不能与gevent共同工作的库结合起来，而不用写哪怕一行代码。 虽然猴子补丁仍然是邪恶的(evil)，但在这种情况下它是“有用的邪恶(useful evil)”。

### 真实世界的应用

Gevent ZeroMQ
ZeroMQ 被它的作者描述为 “一个表现得像一个并发框架的socket库”。 它是一个非常强大的，为构建并发和分布式应用的消息传递层。

ZeroMQ提供了各种各样的socket原语。最简单的是请求-应答socket对 (Request-Response socket pair)。一个socket有两个方法send和recv， 两者一般都是阻塞操作。但是Travis Cline 的一个杰出的库弥补了这一点，这个库使用gevent.socket来以非阻塞的方式 轮询ZereMQ socket。通过命令：

pip install gevent-zeromq

你可以从PyPi安装gevent-zeremq。

```python
# Note: Remember to ``pip install pyzmq gevent_zeromq``
import gevent
from gevent_zeromq import zmq

# Global Context
context = zmq.Context()

def server():
    server_socket = context.socket(zmq.REQ)
    server_socket.bind("tcp://127.0.0.1:5000")

    for request in range(1,10):
        server_socket.send("Hello")
        print('Switched to Server for %s' % request)
        # Implicit context switch occurs here
        server_socket.recv()

def client():
    client_socket = context.socket(zmq.REP)
    client_socket.connect("tcp://127.0.0.1:5000")

    for request in range(1,10):

        client_socket.recv()
        print('Switched to Client for %s' % request)
        # Implicit context switch occurs here
        client_socket.send("World")

publisher = gevent.spawn(server)
client    = gevent.spawn(client)

gevent.joinall([publisher, client])



Switched to Server for 1
Switched to Client for 1
Switched to Server for 2
Switched to Client for 2
Switched to Server for 3
Switched to Client for 3
Switched to Server for 4
Switched to Client for 4
Switched to Server for 5
Switched to Client for 5
Switched to Server for 6
Switched to Client for 6
Switched to Server for 7
Switched to Client for 7
Switched to Server for 8
Switched to Client for 8
Switched to Server for 9
Switched to Client for 9
```


### 参考
* [Gevent指南](http://xlambda.com/gevent-tutorial/)