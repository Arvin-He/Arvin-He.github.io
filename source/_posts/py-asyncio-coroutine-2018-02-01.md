---
title: 关于coroutine
date: 2018-02-01 11:29:08
tags: python
categories: python
---
## 关于协程
协程，又称微线程，纤程。英文名Coroutine。
python中的协程发展有3个阶段:
1. 最初的生成器变形yield/send
2. python3.4引入@asyncio.coroutine和yield from
3. python3.5引入async/await关键字

## 协程库
常用的协程有gevent和asyncio

asyncio是一个基于事件循环的实现异步I/O的模块。

## 从yield说起
当一个函数func中包含yield语句时，python会自动将其识别为一个生成器,
当调用该函数func()时,并**不会真正调用函数体,而是以函数体生成一个生成器对象实例.**
yield在这里保留func函数的计算现场,暂停func的计算并将返回值返回。
而将func放入for...in循环中时，每次循环都会调用next(func())，唤醒生成器，
执行到下一个yield语句处，直到抛出StopIteration异常。此异常会被for循环捕获，导致跳出循环。

## 然后是send
yield实现数据的流出,即产生返回,但是不能接收数据,但如果可以send数据给生成器函数不就相当于实现了协程了嘛.
这时send就起这个发送数据生成器的作用了,这样生成器就能接收数据,再产生数据,形成一个连续的流程了.
于是在python中的生成器有send函数.当一个函数被标记为生成器,那么这个函数就有了send函数了.这是生成器的特性决定的.
Python的生成器不但通过yield可以返回一个值，它还可以接收调用者发出的参数。

## 一个例子
```python
import pymongo
from gevent.pool import Pool
from gevent import monkey
monkey.patch_all()

class TMonitor(object):

    def __init__(self, domain):
        db = ConnectMongo().db
        self.movie = db['xxxxx']
        self.domain = domain
        self.results = []

    def run(self, first, second):
        pool = Pool(10)
        pool.map(self.run_task, self.get_items(first, second))

    def get_item(self, start_end):
        print("~~~~~~~~~~~~~~~~~~~~~")
        select_dict = {'$and': [{'insert_time': {'$gte': start_end[0], '$lte': start_end[1]}}, {'domain': {'$eq': self.domain}}]}
        for item in self.movie.find(select_dict, no_cursor_timeout=True).sort('insert_time', pymongo.ASCENDING):
            insert_time = item['insert_time']
            url = item['request']['url']
            content = item['response']['content']
            task_dict = {'url': url, 'content': content, 'insert_time': insert_time}
            print("ooooooooooooo")
            yield task_dict

    def get_items(self, first, second):
        generator1 = self.get_item(first)
        generator2 = self.get_item(second)
        # 进入生成器
        # next(item1)
        # next(item2)
        generator1.send(None)
        generator2.send(None)
        print('KKKKKKKKKKKKKKKKK')
        while True:
            try:
                item1_res = generator1.send(first)
                item2_res = generator2.send(second)
                yield (item1_res, item2_res)
            except StopIteration:
                break

    def run_task(self, items):
        print('xxxxxxxxxxx')
        pprint(items[0].get('url', 'None'))
        # pprint(items[0].get('insert_time'))
        pprint(items[1].get('url', 'None'))
        # pprint(items[1].get('insert_time'))

if __name__ == "__main__":
    domain = 'www.xxxx.com'
    tamp = TMonitor(domain)
    first = ('1517463871757', '1517463967738')
    second = ('1517464006277', '1517464088805')
    tamp.run(first, second)
```

## yield from来了
yield from用于重构生成器,yield from的作用还体现可以像一个管道一样将send信息传递给内层协程，并且处理好了各种异常情况.

## asyncio.coroutine和yield from
asyncio是一个基于事件循环的实现异步I/O的模块。通过yield from，可以将协程asyncio.sleep(或其他协程)的控制权交给事件循环，
然后挂起当前协程；之后，由事件循环决定何时唤醒asyncio.sleep,接着向后执行代码。

## 关于async和await
async和await两个关键字是python3.5引入的,可以完美替换掉python3.4引入的asyncio.coroutine和yield from.
从Python设计的角度来看，async/await让协程表面上独立于生成器，将细节都隐藏于asyncio模块之下，语法更清晰明了。

## async with 和async for
async with是一个异步上下文管理器,异步上下文管理器指的是在enter和exit方法处能够暂停执行的上下文管理器。
为了实现这样的功能，需要加入两个新的方法：`__aenter__` 和`__aexit__`。这两个方法**都要返回一个 awaitable类型的值**。
异步上下文管理器的一种使用方法是:
```python
class AsyncContextManager:
    async def __aenter__(self):
        await log('entering context')

    async def __aexit__(self, exc_type, exc, tb):
        await log('exiting context')
```
和常规的with表达式一样，可以在一个async with表达式中指定多个上下文管理器。
如果向async with表达式传入的上下文管理器中没有`__aenter__` 和`__aexit__`方法，这将引起一个错误 。
如果在async def函数外面使用async with，将引起一个SyntaxError（语法错误）。
```python

async with async_open('1.txt') as f:
    content = await f.read()
```
相应的，async_open 函数返回的 f 对象需要实现 `__aenter__` 和 `__aexit__` 这 2 个异步方法。

async for是一个异步迭代器,一个异步可迭代对象（asynchronous iterable）能够在迭代过程中调用异步代码，而异步迭代器就是能够在next方法中调用异步代码。为了支持异步迭代：

1. 一个对象必须实现`__aiter__`方法，该方法返回一个异步迭代器（asynchronous iterator）对象。 
2. 一个异步迭代器对象必须实现`__anext__`方法，该方法返回一个awaitable类型的值。 
3. 为了停止迭代，`__anext__`必须抛出一个StopAsyncIteration异常。
异步迭代器的一个例子如下:
```python
class AsyncIterable:
    def __aiter__(self):
        return self

    async def __anext__(self):
        data = await self.fetch_data()
        if data:
            return data
        else:
            raise StopAsyncIteration

    async def fetch_data(self):
        ...
```
```python
f = open('1.txt')
async for line in f:
    print(line)
```
async for 实现了 `__aiter__` 和 `__anext__`方法.
PEP 525 引入的异步生成器（asynchronous generator）就实现了这两个方法。在异步方法中使用 yield 表达式，
会将它变成异步生成器函数（Python 3.6 以后可用，3.5 之前是语法错误）。
值得注意的是，异步生成器没有实现 `__await__` 方法，因此它不是协程，也不能被 await。
把一个没有`__aiter__`方法的迭代对象传递给 async for将引起TypeError。
和常规的for表达式一样， async for也有一个可选的else 分句。

## asyncio使用
协程不能直接运行,需要将协程加入到事件循环(loop)中, `asyncio.get_event_loop`方法可以创建一个事件循环，
然后调用`run_until_complete`将协程注册到事件循环，并启动事件循环。
* event_loop 事件循环：程序开启一个无限的循环，程序员会把一些函数注册到事件循环上。当满足事件发生的时候，调用相应的协程函数。
* coroutine 协程：协程对象，指一个使用async关键字定义的函数，它的调用不会立即执行函数，而是会**返回一个协程对象**。协程对象需要注册到事件循环，由事件循环调用。
* task 任务：一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含任务的各种状态。
* future： 代表将来执行或没有执行的任务的结果。它和task上没有本质的区别
* async/await 关键字：python3.5 用于定义协程的关键字，async定义一个协程，await用于挂起阻塞的异步调用接口。

### 关于task
在注册协程的事件循环的时候，实际上`run_until_complete`方法将协程包装成为了一个任务（task）对象。
而task对象是Future类的子类,保存了协程运行后的状态，用于未来获取协程的结果。
创建task后，task在加入事件循环之前是pending状态，
`asyncio.ensure_future(coroutine)` 和 `loop.create_task(coroutine)`都可以创建一个task，
`run_until_complete`的参数是一个futrue对象。当传入一个协程，其内部会自动封装成task，task是Future的子类。
`isinstance(task, asyncio.Future)`将会输出True。
```python
import asyncio
import time
 
now = lambda: time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
 
start = now()
# 协程对象
coroutine = do_some_work(2)
# 创建事件循环
loop = asyncio.get_event_loop()
# 创建任务task
# task = asyncio.ensure_future(coroutine)
task = loop.create_task(coroutine)

print(task)
# 注册事件,开始事件循环
loop.run_until_complete(task)
print(task)
print('TIME: ', now() - start)
```

### 绑定回调
绑定回调，在task执行完毕的时候可以获取执行的结果，回调的最后一个参数是future对象，
通过该对象可以获取协程返回值。如果回调需要多个参数，可以通过偏函数导入。
```python
import time
import asyncio
 
now = lambda : time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
    return 'Done after {}s'.format(x)
 
def callback(future):
    print('Callback: ', future.result())
 
start = now()
# 创建协程对象
coroutine = do_some_work(2)
# 创建事件循环
loop = asyncio.get_event_loop()
# 创建task
task = asyncio.ensure_future(coroutine)
# 绑定回调
task.add_done_callback(callback)
# 注册事件循环
loop.run_until_complete(task)
 
print('TIME: ', now() - start)
# 回调传入多个参数
def callback(t, future):
    print('Callback:', t, future.result())
# 回调传入多个参数, 通过偏函数导入
task.add_done_callback(functools.partial(callback, 2))
```
可以看到，coroutine执行结束时候会调用回调函数。并通过参数future获取协程执行的结果。
我们创建的task和回调里的future对象，实际上是同一个对象。

### future 与 result
回调一直是很多异步编程的恶梦，程序员更喜欢使用同步的编写方式写异步代码，以避免回调的恶梦。回调中我们使用了future对象的result方法。
前面不绑定回调的例子中，我们可以看到task有finished状态。在那个时候，可以直接读取task的result方法。
```python
async def do_some_work(x):
    print('Waiting {}'.format(x))
    return 'Done after {}s'.format(x)
 
start = now()
 
coroutine = do_some_work(2)
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(coroutine)
loop.run_until_complete(task)
 
print('Task ret: {}'.format(task.result()))
print('TIME: {}'.format(now() - start))
```
可以看到输出的结果：
```
Waiting:  2
Task ret:  Done after 2s
TIME:  0.0003650188446044922
```

### 阻塞和await
使用async可以定义协程对象，使用await可以针对耗时的操作进行挂起，就像生成器里的yield一样，函数让出控制权。
协程遇到await，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行。
耗时的操作一般是一些IO操作，例如网络请求，文件读取等。使用asyncio.sleep函数来模拟IO操作。协程的目的也是让这些IO操作异步化。
**注意:**asyncio.sleep本身也是一个协程函数,而time.sleep则不是协程函数,而且是阻塞的.
```python
import asyncio
import time
 
now = lambda: time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)
 
start = now()
 
coroutine = do_some_work(2)
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(coroutine)
loop.run_until_complete(task)
 
print('Task ret: ', task.result())
print('TIME: ', now() - start)
```
在 sleep的时候，使用await让出控制权。即当遇到阻塞调用的函数的时候，使用await方法将协程的控制权让出，以便loop调用其他的协程。
现在我们的例子就用耗时的阻塞操作了。

### 并行和并发
并发和并行一直是容易混淆的概念。
1. 并发通常指: 有多个任务需要同时进行，
2. 并行则是: 同一时刻有多个任务执行。
用上课来举例就是，并发情况下是一个老师在同一时间段辅助不同的人功课。并行则是好几个老师分别同时辅助多个学生功课。
简而言之就是一个人同时吃三个馒头还是三个人同时分别吃一个的情况，吃一个馒头算一个任务。
asyncio实现并发，就需要多个协程来完成任务，每当有任务阻塞的时候就await，然后其他协程继续工作。
创建多个协程的列表，然后将这些协程注册到事件循环中。
```python
import asyncio
 
import time
 
now = lambda: time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)
 
start = now()
 
coroutine1 = do_some_work(1)
coroutine2 = do_some_work(2)
coroutine3 = do_some_work(4)
 
tasks = [
    asyncio.ensure_future(coroutine1),
    asyncio.ensure_future(coroutine2),
    asyncio.ensure_future(coroutine3)
]
 
loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))
 
for task in tasks:
    print('Task ret: ', task.result())
 
print('TIME: ', now() - start)
```
结果如下:
```
Waiting:  1
Waiting:  2
Waiting:  4
Task ret:  Done after 1s
Task ret:  Done after 2s
Task ret:  Done after 4s
TIME:  4.003541946411133
```
总时间为4s左右。4s的阻塞时间，足够前面两个协程执行完毕。
如果是同步顺序的任务，那么至少需要7s。此时我们使用了aysncio实现了并发。
asyncio.wait(tasks) 也可以使用 asyncio.gather(*tasks) ,前者接受一个task列表，后者接收一堆task。

### 协程嵌套
使用async可以定义协程，协程用于耗时的io操作，我们也可以封装更多的io操作过程，
这样就实现了嵌套的协程，即一个协程中await了另外一个协程，如此连接起来。

在main协程函数里处理结果
```python
import asyncio
import time
now = lambda: time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)
 
async def main():
    coroutine1 = do_some_work(1)
    coroutine2 = do_some_work(2)
    coroutine3 = do_some_work(4)
    tasks = [
        asyncio.ensure_future(coroutine1),
        asyncio.ensure_future(coroutine2),
        asyncio.ensure_future(coroutine3)
    ]
    # 方式一
    dones, pendings = await asyncio.wait(tasks)
    for task in dones:
        print('Task ret: ', task.result())
    # 方式二:使用asyncio.gather创建协程对象，那么await的返回值就是协程运行的结果。
    results = await asyncio.gather(*tasks)
    for result in results:
        print('Task ret: ', result)
    
start = now()
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
print('TIME: ', now() - start)
```

不在main协程函数里处理结果，直接返回await的内容，那么最外层的`run_until_complete`将会返回main协程的结果。
```python
async def main():
    coroutine1 = do_some_work(1)
    coroutine2 = do_some_work(2)
    coroutine3 = do_some_work(2)
    tasks = [
        asyncio.ensure_future(coroutine1),
        asyncio.ensure_future(coroutine2),
        asyncio.ensure_future(coroutine3)
    ]
    # 不在main协程函数里处理结果，直接返回await的内容
    # 方式一
    return await asyncio.gather(*tasks)
    # 方式二: 返回使用asyncio.wait方式挂起协程。
    return await asyncio.wait(tasks)
    # 方式三: 使用asyncio的as_completed方法
    for task in asyncio.as_completed(tasks):
        result = await task
        print('Task ret: {}'.format(result))
 
start = now()
loop = asyncio.get_event_loop()
# 这里获取main协程的结果
# 方式一处理结果
results = loop.run_until_complete(main())
for result in results:
    print('Task ret: ', result)
# 方式二处理结果
done, pending = loop.run_until_complete(main())
for task in done:
    print('Task ret: ', task.result())
# 方式三处理结果
done = loop.run_until_complete(main())
print('TIME: ', now() - start)
```
由此可见，协程的调用和组合十分灵活，尤其是对于结果的处理，如何返回，如何挂起，需要逐渐积累经验和前瞻的设计。

### 协程停止
以上是协程的几种常用的用法，都是协程围绕着**事件循环**进行的操作。future对象有几个状态：
* Pending
* Running
* Done
* Cancelled
创建future的时候，task为pending，
事件循环调用执行的时候是running，
调用完毕就是done，
如果需要停止事件循环，就需要先把task取消。可以使用asyncio.Task获取事件循环的task
```python
import asyncio
import time
now = lambda: time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)
 
coroutine1 = do_some_work(1)
coroutine2 = do_some_work(2)
coroutine3 = do_some_work(2)
 
tasks = [
    asyncio.ensure_future(coroutine1),
    asyncio.ensure_future(coroutine2),
    asyncio.ensure_future(coroutine3)
]
 
start = now()
loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(asyncio.wait(tasks))
except KeyboardInterrupt as e:
    # print(asyncio.Task.all_tasks())
    for task in asyncio.Task.all_tasks():
        print(task.cancel())
    loop.stop()
    loop.run_forever()
finally:
    loop.close()
 
print('TIME: ', now() - start)
```
启动事件循环之后，马上ctrl+c，会触发`run_until_complete`的执行异常 KeyBorardInterrupt。
然后通过循环asyncio.Task取消future。可以看到输出如下：
```
Waiting:  1
Waiting:  2
Waiting:  2
True
True
True
True
TIME:  0.8858370780944824
```
True表示cannel成功，loop stop之后还需要再次开启事件循环，最后在close，不然还会抛出异常：
```
Task was destroyed but it is pending!
task: <Task pending coro=<do_some_work() done,
```
循环task，逐个cancel是一种方案，可是正如上面我们把task的列表封装在main函数中，main函数外进行事件循环的调用。
这个时候，main相当于最外出的一个task，那么处理包装的main函数即可。
```python
import asyncio
import time
now = lambda: time.time()
 
async def do_some_work(x):
    print('Waiting: ', x)
 
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)
 
async def main():
    coroutine1 = do_some_work(1)
    coroutine2 = do_some_work(2)
    coroutine3 = do_some_work(2)
 
    tasks = [
        asyncio.ensure_future(coroutine1),
        asyncio.ensure_future(coroutine2),
        asyncio.ensure_future(coroutine3)
    ]
    done, pending = await asyncio.wait(tasks)
    for task in done:
        print('Task ret: ', task.result())
 
start = now()
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(main())
try:
    loop.run_until_complete(task)
except KeyboardInterrupt as e:
    print(asyncio.Task.all_tasks())
    print(asyncio.gather(*asyncio.Task.all_tasks()).cancel())
    loop.stop()
    loop.run_forever()
finally:
    loop.close()
```
### 不同线程的事件循环
很多时候，我们的事件循环用于注册协程，而有的协程需要动态的添加到事件循环中。
一个简单的方式就是使用多线程: **当前线程创建**一个事件循环，然后在**新建一个线程**，在**新线程中启动事件循环**。当前线程不会被block。
```python
from threading import Thread
 
def start_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()
 
def more_work(x):
    print('More work {}'.format(x))
    time.sleep(x)
    print('Finished more work {}'.format(x))
 
start = now()
new_loop = asyncio.new_event_loop()
# 创建新线程,在新线程中启动事件循环
t = Thread(target=start_loop, args=(new_loop,))
t.start()
print('TIME: {}'.format(time.time() - start))
 
new_loop.call_soon_threadsafe(more_work, 6)
new_loop.call_soon_threadsafe(more_work, 3)
```
启动上述代码之后，当前线程不会被block，新线程中会按照顺序执行`call_soon_threadsafe`方法注册的`more_work`方法，
后者因为time.sleep操作是同步阻塞的，因此运行完毕more_work需要大致6 + 3

改进:新线程协程
```python
def start_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()
 
async def do_some_work(x):
    print('Waiting {}'.format(x))
    await asyncio.sleep(x)
    print('Done after {}s'.format(x))
 
def more_work(x):
    print('More work {}'.format(x))
    time.sleep(x)
    print('Finished more work {}'.format(x))
 
start = now()
# 主线程创建一个new_loop
new_loop = asyncio.new_event_loop()
# 子线程中启动这个事件循环
t = Thread(target=start_loop, args=(new_loop,))
t.start()
print('TIME: {}'.format(time.time() - start))
 
asyncio.run_coroutine_threadsafe(do_some_work(6), new_loop)
asyncio.run_coroutine_threadsafe(do_some_work(4), new_loop)
```
上述的例子，主线程中创建一个`new_loop`，然后在另外的子线程中开启一个无限事件循环。
主线程通过`run_coroutine_threadsafe`新注册协程对象。
这样就能在子线程中进行事件循环的并发操作，同时主线程又不会被block。一共执行的时间大概在6s左右。

### master-worker主从模式
对于并发任务，通常是用生成消费模型，对队列的处理可以使用类似master-worker的方式，master主要用户获取队列的msg，worker用户处理消息。
并且协程更适合单线程的方式, 为了简单起见，我们的主线程用来监听队列，子线程用于处理队列。这里使用redis的队列。主线程中有一个是无限循环，用户消费队列。
```python
 while True:
    task = rcon.rpop("queue")
    if not task:
        time.sleep(1)
        continue
    asyncio.run_coroutine_threadsafe(do_some_work(int(task)), new_loop)
```

### 停止子线程
如果一切正常，那么上面的例子很完美。可是，需要停止程序，直接ctrl+c，会抛出KeyboardInterrupt错误，我们修改一下主循环：
```python
try:
    while True:
        task = rcon.rpop("queue")
        if not task:
            time.sleep(1)
            continue
        asyncio.run_coroutine_threadsafe(do_some_work(int(task)), new_loop)
except KeyboardInterrupt as e:
    print(e)
    new_loop.stop()
```
可是实际上并不好使，虽然主线程try了KeyboardInterrupt异常，但是子线程并没有退出，
为了解决这个问题，可以设置子线程为守护线程，这样当主线程结束的时候，子线程也随之退出。
```python
new_loop = asyncio.new_event_loop()
t = Thread(target=start_loop, args=(new_loop,))
# 设置子线程为守护线程
t.setDaemon(True)
t.start()
 
try:
    while True:
        # print('start rpop')
        task = rcon.rpop("queue")
        if not task:
            time.sleep(1)
            continue
        asyncio.run_coroutine_threadsafe(do_some_work(int(task)), new_loop)
except KeyboardInterrupt as e:
    print(e)
    new_loop.stop()
```

### 关于队列的task_done, join
q.task_done(): 每次从queue中get一个数据之后，当处理好相关问题后，**才调用该方法**，以提示q.join()是否停止阻塞，让线程向前执行或者退出；
如果每从队列里取一次，但没有执行`task_done()`，则join无法判断队列到底有没有结束，在最后执行个join()是等不到结果的，会一直挂起。
可以理解为，每`task_done`一次 就从队列里删掉一个元素，这样在最后join的时候根据队列长度是否为零来判断队列是否结束，从而执行主线程。
q.join(): 阻塞，直到queue中的数据均被删除或者处理。为队列中的每一项都调用一次。
对于生产者-消费者模型，这样做还是有问题的，因为如果queue初始为空，q.join()会直接停止阻塞，继而执行后续语句；
如果有多个消费者，没有生产者，且queue始初化为一定的数据量，则可以正常执行。

注意点: 
1. put队列完成的时候千万不能用`task_done()`，否则会报错：`task_done() called too many times`, 因为该方法仅仅表示get成功后，执行的一个标记。

### 关于守护线程和线程的join
threading：
守护线程不同于linux中守护进程的概念
t.setDaemon(True)  将线程设置成守护线程，主进行结束后，此线程也会被强制结束。如果线程没有设置此值，则主线程执行完毕后还会等待此线程执行。

t.join() 线程阻塞，只有当线程运行结束后才会继续执行后续语句


## 参考
* [Python黑魔法---异步IO（ asyncio）协程](http://python.jobbole.com/87310/)
* [Python异步处理中的async with 和async for 用法说明](http://blog.csdn.net/qq_35304570/article/details/78209612)