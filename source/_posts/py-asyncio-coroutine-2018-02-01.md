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

## asyncio使用

## 队列的使用

### 关于task_done, join

## gevent使用

