---
title: Python之asyncio使用
date: 2017-06-06 16:00:58
tags: python
categories: python
---
### asyncio简介
asyncio是Python 3.4版本引入的标准库，直接内置了对异步IO的支持。
asyncio的编程模型就是一个消息循环。我们从asyncio模块中直接获取一个EventLoop的引用，然后把需要执行的协程扔到EventLoop中执行，就实现了异步IO。

到了Python 3.5添加了async和await两个关键字，分别用来替换asyncio.coroutine和yield from。自此，协程成为新的语法，而不再是一种生成器类型了。此外,事件循环与协程的引入，可以极大提高高负载下程序的I/O性能。除此之外还增加了async with(异步上下文管理)、async for(异步迭代器)语法。还有在新发布的Python 3.6里面可以用异步生成器了.

当给一个函数添加了async关键字，就会把它变成一个异步函数。
async/await是Python提供的异步编程API，而asyncio只是一个利用 async/await API进行异步编程的框架.
现存的一些库其实并不能原生的支持asyncio（因为会发生阻塞或者功能不可用），比如requests，
如果要写爬虫，配合asyncio的应该用aiohttp，其他的如数据库驱动等各种Python对应的库也都得使用对应的aioXXX版本了。

需要进行协程切换的地方，就需要使用await关键字

### 同步与异步/阻塞与非阻塞概念
#### 同步与异步 
同步和异步关注的是消息通信机制 (synchronous communication/ asynchronous communication)
所谓同步，就是在发出一个调用时，在没有得到结果之前，该调用就不返回。但是一旦调用返回，就得到返回值了。
换句话说，就是由调用者主动等待这个调用的结果。
而异步则是相反，调用在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在调用发出后，被调用者通过状态、通知来通知调用者，或通过回调函数处理这个调用。
典型的异步编程模型比如Node.js，举个通俗的例子：
你打电话问书店老板有没有《分布式系统》这本书，如果是同步通信机制，书店老板会说，你稍等，”我查一下"，然后开始查啊查，等查好了（可能是5秒，也可能是一天）告诉你结果（返回结果）。
而异步通信机制，书店老板直接告诉你我查一下啊，查好了打电话给你，然后直接挂电话了（不返回结果）。然后查好了，他会主动打电话给你。在这里老板通过“回电”这种方式来回调。

#### 阻塞与非阻塞
阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态.
阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。
还是上面的例子，
你打电话问书店老板有没有《分布式系统》这本书，你如果是阻塞式调用，你会一直把自己“挂起”，直到得到这本书有没有的结果，如果是非阻塞式调用，你不管老板有没有告诉你，你自己先一边去玩了， 当然你也要偶尔过几分钟check一下老板有没有返回结果。
在这里阻塞与非阻塞与是否同步异步无关。跟老板通过什么方式回答你结果无关。

### asyncio使用
```python
import asyncio
# 将heloo
@asyncio.coroutine
def hello():
    print("Hello world!")
    # 异步调用asyncio.sleep(1):
    r = yield from asyncio.sleep(1)
    print("Hello again!")

# 获取EventLoop:
loop = asyncio.get_event_loop()
# 执行coroutine
loop.run_until_complete(hello())
loop.close()
```

`@asyncio.coroutine`把一个generator标记为coroutine类型，然后，我们就把这个coroutine扔到EventLoop中执行。

`hello()`会首先打印出Hello world!，然后，`yield from`语法可以让我们方便地调用另一个generator。由于`asyncio.sleep()`也是一个coroutine，所以线程不会等待`asyncio.sleep()`，而是直接中断并执行下一个消息循环。当`asyncio.sleep()`返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。

把asyncio.sleep(1)看成是一个耗时1秒的IO操作，在此期间，主线程并未等待，而是去执行EventLoop中其他可以执行的coroutine了，因此可以实现并发执行。

我们用Task封装两个coroutine试试：
```python
import threading
import asyncio

@asyncio.coroutine
def hello1():
    print('Hello 1 in! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1)
    print('Hello 1 out! (%s)' % threading.currentThread())

    
@asyncio.coroutine
def hello2():
    print('Hello 2 in! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1)
    print('Hello 2 out! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello1(), hello2()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()


Hello 1 in! (<_MainThread(MainThread, started 6620)>)
Hello 2 in! (<_MainThread(MainThread, started 6620)>)
Hello 1 out! (<_MainThread(MainThread, started 6620)>)
Hello 2 out! (<_MainThread(MainThread, started 6620)>)
```

由打印的当前线程名称可以看出，两个coroutine是由同一个线程并发执行的。
如果把asyncio.sleep()换成真正的IO操作，则多个coroutine就可以由一个线程并发执行。

### async/await
Python 3.5开始引入了新的语法async和await，可以让coroutine的代码更简洁易读。
**请注意:**async和await是针对coroutine的新语法,要使用新语法,只需要做两步简单的替换：
1. 把@asyncio.coroutine替换为async；
2. 把yield from替换为await。

```python
@asyncio.coroutine
def hello():
    print("Hello world!")
    r = yield from asyncio.sleep(1)
    print("Hello again!")
```

async/await新语法
```python
async def hello():
    print("Hello world!")
    r = await asyncio.sleep(1)
    print("Hello again!")
```


### 参考
* [https://juejin.im/post/5857b8a98e450a006cb060fc](https://juejin.im/post/5857b8a98e450a006cb060fc)
* [廖雪峰教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00144661533005329786387b5684be385062a121e834ac7000)