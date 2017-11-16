---
title: tornado学习笔记(二)
date: 2017-11-16 21:40:47
tags: tornado
categories: python
---

### 一些说明
ornado 所谓的异步：就是你调用我之后，我发现数据没准备好，那我就不处理，而是跳到程序的其他地方继续执行，等数据准备好之后再切回来继续执行。Tornado 的 IOLoop 就是一个总调度器，汇总了所有的 events 和 callbacks，然后同步执行。这会整体生提升性能，但不会降低单个请求的响应时间。


## 一些接口说明
### tornado.web.asynchronous
其中 tornado.web.asynchronous 装饰器很简单，就是设置 `self._auto_finish = False`，
这样当 AsyncHandler.get() 执行完之后，connection socket 不会被 close，需要主动调用 self.finish()。
在保持连接不关闭的情况下，把控制权让出去，等数据就绪之后再切回来，使异步实现成为可能。
```python
class AsyncHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    def get(self):
        http_client = AsyncHTTPClient()
        http_client.fetch("http://example.com",
                          callback=self.on_fetch)

    def on_fetch(self, response):
        self.write("Downloaded!")
        self.finish()
```

### SimpleAsyncHTTPClient
AsyncHTTPClient 的处理流程，简单概括就是：与 HTTP Server 建立连接，等拿到 response 后再来调用回调函数 on_fetch。首先通过创建非阻塞的 socket 连接，然后放入到 ioloop 中，当数据可写/可读之后再接着处理。

### gen.engine 的实现：
gen.engine 的作用就是把异步中 callback 的写法通过 yield 替代。
以下的代码分析都是基于 Tornado 3.0。
tornado.ioloop.IOLoop.current() 创建一个ioloop实例

###IOLoop 为何没有 `__init__`函数
其实是因为要初始化成为单例，IOLoop 的 new 函数已经被改写了，同时指定了 initialize 做为它的初始化方法，所以没有 `__init__` 。 



### Future
Future在tornado中是一个很奇妙的对象，它是一个穿梭于协程和调度器之间的信使。提供了回调函数注册(当异步事件完成后，调用注册的回调)、中间结果保存、嵌套协程唤醒父协程(通过Runner实现)等功能。Coroutine和Future是一一对应的，可以从上节gen.coroutine装饰器的实现中看到。每调用一个协程，表达式所返回的就是一个Future对象，它所表达的意义为：这个协程的内部各种异步逻辑执行完毕后，会把结果保存在这个Future中，同时调用这个Future中指定的回调函数，而future中的回调函数是什么时候被注册的呢？那就是当前——你通过调用协程，返回了这个future对象的时候：



