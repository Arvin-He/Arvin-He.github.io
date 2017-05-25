---
title: Python之WSGI
date: 2017-05-25 13:17:45
tags: Python,Web
categories: 编程
---
### WSGI概念
我们先看一下面向 http 的 python 程序需要关心哪些内容：

* 请求
    * 请求的方法 method
    * 请求的地址 url
    * 请求的内容
    * 请求的头部 header
    * 请求的环境信息
* 响应
    * 状态码 status_code
    * 响应的数据
    * 响应的头部

WSGI（Web Server Gateway Interface） 的任务就是把上面的数据在 http server 和 python 应用程序之间简单友好地传递。
它是一个标准，被定义在PEP 333。需要 http server 和 python 应用程序都要遵守一定的规范，实现这个标准的约定内容，才能正常工作。

### 应用程序端
WSGI 规定每个 python 程序（Application）必须是一个可调用的对象（实现了\_\_call\_\_ 函数的方法或者类），
接受两个参数 environ（WSGI 的环境信息） 和 start_response（开始响应请求的函数），并且返回 iterable。

几点说明：
* environ 和 start_response 由 http server 提供并实现
* environ 变量是包含了环境信息的字典
* Application 内部在返回前调用 start_response
* start\_response也是一个 callable，接受两个必须的参数，status（HTTP状态）和 response\_headers（响应消息的头）
* 可调用对象要返回一个值，这个值是可迭代的。

```python
 # 1. 可调用对象是一个函数,函数都是可调用的
def application(environ, start_response):
   response_body = 'The request method was %s' % environ['REQUEST_METHOD']
   # HTTP response code and message
   status = '200 OK'
   # 应答的头部是一个列表，每对键值都必须是一个 tuple。
   response_headers = [('Content-Type', 'text/plain'),
                       ('Content-Length', str(len(response_body)))]
   # 调用服务器程序提供的 start_response，填入两个参数
   start_response(status, response_headers)
   # 返回必须是 iterable
   return [response_body]	
   
# 2. 可调用对象是一个类
class AppClass:
	"""这里的可调用对象就是 AppClass 这个类，调用它就能生成可以迭代的结果。
		使用方法类似于： 
		for result in AppClass(env, start_response):
		     do_somthing(result)
	"""
    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"

# 3. 可调用对象是一个实例 
class AppClass:
	"""这里的可调用对象就是 AppClass 的实例，使用方法类似于： 
		app = AppClass()
		for result in app(environ, start_response):
		     do_somthing(result)
	"""
    def __init__(self):
        pass

    def __call__(self, environ, start_response):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
```

### 服务器程序端
标准要能够确切地实行，必须要求程序端和服务器端共同遵守。
上面提到， envrion 和 start_response 都是服务器端提供的。

下面就看看，服务器端要履行的义务:
准备 environ 参数
定义 start_response 函数
调用程序端的可调用对象

PEP 333 里给出了一个 wsgi server 的简单实现，我又简化了一下——去除一些异常处理和判断，添加了一点注释：
```python
import os, sys

def run_with_cgi(application):    # application 是程序端的可调用对象
	# 准备 environ 参数，这是一个字典，里面的内容是一次 HTTP 请求的环境变量
    environ = dict(os.environ.items())
    environ['wsgi.input']        = sys.stdin
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True	        
    environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

	# 把应答的结果输出到终端
    def write(data):
        sys.stdout.write(data)
        sys.stdout.flush()

	# 实现 start_response 函数，根据程序端传过来的 status 和 response_headers 参数，
	# 设置状态和头部
    def start_response(status, response_headers, exc_info=None):
        headers_set[:] = [status, response_headers]
      	return write

	# 调用客户端的可调用对象，把准备好的参数传递过去
    result = application(environ, start_response)
    
    # 处理得到的结果，这里简单地把结果输出到标准输出。
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
    finally:
        if hasattr(result, 'close'):
            result.close()
```

### 中间层 middleware
有些程序可能处于服务器端和程序端两者之间：对于服务器程序，它就是应用程序；而对于应用程序，它就是服务器程序。这就是中间层 middleware。middleware 对服务器程序和应用是透明的，它像一个代理/管道一样，把接收到的请求进行一些处理，然后往后传递，一直传递到客户端程序，最后把程序的客户端处理的结果再返回。

middleware 做了两件事情：
* 被服务器程序（有可能是其他 middleware）调用，返回结果回去
* 调用应用程序（有可能是其他 middleware），把参数传递过去

PEP 333 上面给出了 middleware 的可能使用场景：
* 根据 url 把请求给到不同的客户端程序（url routing）
* 允许多个客户端程序/web 框架同时运行，就是把接到的同一个请求传递给多个程序。
* 负载均衡和远程处理：把请求在网络上传输
* 应答的过滤处理

那么简单地 middleware 实现是怎么样的呢？下面的代码实现的是一个简单地 url routing 的 middleware：
```python
class Router(object):
    def __init__(self):
        self.path_info = {}
    def route(self, environ, start_response):
        application = self.path_info[environ['PATH_INFO']]
        return application(environ, start_response)
    def __call__(self, path):
        def wrapper(application):
            self.path_info[path] = application
        return wrapper

router = Router()
```
怎么在程序里面使用呢？
```python
#here is the application
@router('/hello')	#调用 route 实例，把函数注册到 paht_info 字典
def hello(environ, start_response):
    status = '200 OK'
    output = 'Hello'
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    write = start_response(status, response_headers)
    return [output]

@router('/world')
def world(environ, start_response):
    status = '200 OK'
    output = 'World!'
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    write = start_response(status, response_headers)
    return [output]

#here run the application
result = router.route(environ, start_response)
for value in result: 
    write(value)
```

### 想要更多的话，就去看 PEP333，文档里还有下面更多的知识：
* 错误处理
* environ 变量包含哪些值，都是什么意思。
* 输入和输出的细节
* start_response 的更多规范
* content-length 等头部规范
* 缓存和文本流
* unicode 和多语言处理
* ……

### 参考
* [官方文档 PEP333](http://legacy.python.org/dev/peps/pep-0333/#rationale-and-goals)
* [Getting Started with WSGI](http://lucumr.pocoo.org/2007/5/21/getting-started-with-wsgi/)
* [http://cizixs.com/2014/11/08/understand-wsgi](http://cizixs.com/2014/11/08/understand-wsgi)
* [http://blog.csdn.net/yangz_xx/article/details/37508909](http://blog.csdn.net/yangz_xx/article/details/37508909)
* [wsgiref：官方的 wsgi 实现，包括客户端和服务器端](https://docs.python.org/2/library/wsgiref.html)