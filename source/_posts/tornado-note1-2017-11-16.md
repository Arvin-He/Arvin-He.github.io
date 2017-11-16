---
title: tornado学习笔记(一)
date: 2017-11-16 19:42:00
tags: tornado
categories: python
---
### 说明
高性能源于Tornado基于Epoll（unix为kqueue）的异步网络IO。因为tornado的单线程机制，一不小心就容易写出阻塞服务（block）的代码。不但没有性能提高，反而会让性能急剧下降。因此，探索tornado的异步使用方式很有必要。
tornado虽然以异步非阻塞高性能著称的框架,但默认不是异步非阻塞的,你需要添加两个装饰器@tornado.web.asynchronous和@tornado.gen.coroutine,
这样才算是异步非阻塞的.然而当你用tornado作为http服务器,同时用了requests库发送请求后,你会发现请求还是一个一个的处理,根本不是异步非阻塞的.
原因是requests库发送请求是阻塞的,它会阻塞整个tornado服务进程,为什么阻塞,原因是用 requests 的话能会 block 其他 http 请求。因为他无法注册到 ioloop 里面。只要这个 requests 请求结束， tornado 才会处理下一个 http 请求。 
### Tornado主要模块
web - FriendFeed 使用的基础 Web 框架，包含了 Tornado 的大多数重要的功能
escape - XHTML, JSON, URL 的编码/解码方法
database - 对 MySQLdb 的简单封装，使其更容易使用
template - 基于 Python 的 web 模板系统
httpclient - 非阻塞式 HTTP 客户端，它被设计用来和 web 及 httpserver 协同工作
auth - 第三方认证的实现（包括 Google OpenID/OAuth、Facebook Platform、Yahoo BBAuth、FriendFeed OpenID/OAuth、Twitter OAuth）
locale - 针对本地化和翻译的支持
options - 命令行和配置文件解析工具，针对服务器环境做了优化
 
底层模块
httpserver - 服务于 web 模块的一个非常简单的 HTTP 服务器的实现
iostream - 对非阻塞式的 socket 的简单封装，以方便常用读写操作
ioloop - 核心的 I/O 循环

### Tornado 异步使用方式
简而言之，Tornado的异步包括两个方面，异步服务端和异步客户端。无论服务端和客户端，具体的异步模型又可以分为回调（callback）和协程（coroutine）。具体应用场景，也没有很明确的界限。往往一个请求服务里还包含对别的服务的客户端异步请求。

### 服务端异步方式
服务端异步，可以理解为一个tornado请求之内，需要做一个耗时的任务。直接写在业务逻辑里可能会block整个服务。因此可以把这个任务放到异步处理，实现异步的方式就有两种，
一种是yield挂起函数，另外一种就是使用类线程池的方式。请看一个同步例子：
同步代码:
```python
class SyncHandler(tornado.web.RequestHandler):
  def get(self, *args, **kwargs):
    # 耗时的代码
    os.system("ping -c 2 www.google.com")
    self.finish('It works')
```
测试一下：0.99，姑且当成每秒处理一个请求吧。

异步代码:
```python
class AsyncHandler(tornado.web.RequestHandler):

  @tornado.web.asynchronous
  @tornado.gen.coroutine
  def get(self, *args, **kwargs):
    tornado.ioloop.IOLoop.instance().add_timeout(1, callback=functools.partial(self.ping, 'www.google.com'))
    # do something others
    self.finish('It works')

  @tornado.gen.coroutine
  def ping(self, url):
    os.system("ping -c 2 {}".format(url))
    return 'after'

```
尽管在执行异步任务的时候选择了timeout 1秒，主线程的返回还是很快的。
上述的使用方式，通过tornado的IO循环，把可以把耗时的任务放到后台异步计算，请求可以接着做别的计算。
可是，经常有一些耗时的任务完成之后，我们需要其计算的结果。此时这种方式就不行了。车道山前必有路，只需要切换一异步方式即可。下面使用协程来改写：
```python
class AsyncTaskHandler(tornado.web.RequestHandler):

    @tornado.web.asynchronous
    @tornado.gen.coroutine
    def get(self, *args, **kwargs):
        # yield 结果
        response = yield tornado.gen.Task(self.ping, ' www.google.com')
        print 'response', response
        self.finish('hello')

    @tornado.gen.coroutine
    def ping(self, url):
        os.system("ping -c 2 {}".format(url))
        return 'after'
```    
可以看到异步在处理，而结果值也被返回了。有时候这种协程处理，未必就比同步快。在并发量很小的情况下，IO本身拉开的差距并不大。甚至协程和同步性能差不多。
yield挂起函数协程，尽管没有block主线程，因为需要处理返回值，挂起到响应执行还是有时间等待，相对于单个请求而言。另外一种使用异步和协程的方式就是在主线程之外，使用线程池，线程池依赖于futures。Python2需要额外安装。
```python
下面使用线程池的方式修改为异步处理：
from concurrent.futures import ThreadPoolExecutor

class FutureHandler(tornado.web.RequestHandler):
  executor = ThreadPoolExecutor(10)

  @tornado.web.asynchronous
  @tornado.gen.coroutine
  def get(self, *args, **kwargs):
    url = 'www.google.com'
    tornado.ioloop.IOLoop.instance().add_callback(functools.partial(self.ping, url))
    self.finish('It works')

  @tornado.concurrent.run_on_executor
  def ping(self, url):
    os.system("ping -c 2 {}".format(url))
```    
再切换一下使用方式接口。使用tornado的gen模块下的with_timeout功能（这个功能必须在tornado>3.2的版本）。
```python
class Executor(ThreadPoolExecutor):
  _instance = None

  def __new__(cls, *args, **kwargs):
    if not getattr(cls, '_instance', None):
      cls._instance = ThreadPoolExecutor(max_workers=10)
    return cls._instance


class FutureResponseHandler(tornado.web.RequestHandler):
  executor = Executor()

  @tornado.web.asynchronous
  @tornado.gen.coroutine
  def get(self, *args, **kwargs):
    future = Executor().submit(self.ping, 'www.google.com')
    response = yield tornado.gen.with_timeout(datetime.timedelta(10), future,
                         quiet_exceptions=tornado.gen.TimeoutError)
    if response:
      print 'response', response.result()

  @tornado.concurrent.run_on_executor
  def ping(self, url):
    os.system("ping -c 1 {}".format(url))
    return 'after'
```
线程池的方式也可以通过使用tornado的yield把函数挂起，实现了协程处理。可以得出耗时任务的result，同时不会block住主线程。


###异步多样化
Tornado异步服务的处理大抵如此。现在异步处理的框架和库也很多，借助redis或者celery等，也可以把tonrado中一些业务异步化，放到后台执行。
此外，Tornado还有客户端异步功能。该特性主要是在于 AsyncHTTPClient的使用。此时的应用场景往往是tornado服务内，需要针对另外的IO进行请求和处理。顺便提及，上述的例子中，调用ping其实也算是一种服务内的IO处理。接下来，将会探索一下AsyncHTTPClient的使用，尤其是使用AsyncHTTPClient上传文件与转发请求。
### 异步客户端
前面了解Tornado的异步任务的常用做法，姑且归结为异步服务。通常在我们的服务内，还需要异步的请求第三方服务。针对HTTP请求，Python的库Requests是最好用的库，没有之一。官网宣称：HTTP for Human。然而，在tornado中直接使用requests将会是一场恶梦。requests的请求会block整个服务进程。
上帝关上门的时候，往往回打开一扇窗。Tornado提供了一个基于框架本身的异步HTTP客户端（当然也有同步的客户端）--- AsyncHTTPClient。
### AsyncHTTPClient 基本用法
AsyncHTTPClient是 tornado.httpclinet 提供的一个异步http客户端。使用也比较简单。与服务进程一样，AsyncHTTPClient也可以callback和yield两种使用方式。前者不会返回结果，后者则会返回response。如果请求第三方服务是同步方式，同样会杀死性能。
```python
class SyncHandler(tornado.web.RequestHandler):
  
  def get(self, *args, **kwargs):
    url = 'https://api.github.com/'
    resp = requests.get(url)
    print resp.status_code
    self.finish('It works')
```
性能相当慢了，换成AsyncHTTPClient再测：

```python
class AsyncHandler(tornado.web.RequestHandler):
  
  @tornado.web.asynchronous
  def get(self, *args, **kwargs):
    url = 'https://api.github.com/'
    http_client = tornado.httpclient.AsyncHTTPClient()
    http_client.fetch(url, self.on_response)
    self.finish('It works')

  @tornado.gen.coroutine
  def on_response(self, response):
    print response.code
```

提高了很多,同样，为了获取response的结果，只需要yield函数。
```python
class AsyncResponseHandler(tornado.web.RequestHandler):
  
  @tornado.web.asynchronous
  @tornado.gen.coroutine
  def get(self, *args, **kwargs):
    url = 'https://api.github.com/'
    http_client = tornado.httpclient.AsyncHTTPClient()
    response = yield tornado.gen.Task(http_client.fetch, url)
    print response.code
    print response.body
```

### AsyncHTTPClient 转发
使用Tornado经常需要做一些转发服务，需要借助AsyncHTTPClient。既然是转发，就不可能只有get方法，post，put，delete等方法也会有。此时涉及到一些 headers和body，甚至还有https的waring。

下面请看一个post的例子， yield结果，通常，使用yield的时候，handler是需要 tornado.gen.coroutine。
```python
headers = self.request.headers
body = json.dumps({'name': 'rsj217'})
http_client = tornado.httpclient.AsyncHTTPClient()

resp = yield tornado.gen.Task(
  self.http_client.fetch, 
  url,
  method="POST", 
  headers=headers,
  body=body, 
  validate_cert=False)
```

### AsyncHTTPClient 构造请求
如果业务处理并不是在handlers写的，而是在别的地方，当无法直接使用tornado.gen.coroutine的时候，可以构造请求，使用callback的方式。
```python
body = urllib.urlencode(params)
req = tornado.httpclient.HTTPRequest(
 url=url, 
 method='POST', 
 body=body, 
 validate_cert=False) 

http_client.fetch(req, self.handler_response)
def handler_response(self, response):
  print response.code
```

用法也比较简单，AsyncHTTPClient中的fetch方法，第一个参数其实是一个HTTPRequest实例对象，因此对于一些和http请求有关的参数，例如method和body，可以使用HTTPRequest先构造一个请求，再扔给fetch方法。通常在转发服务的时候，如果开起了validate_cert，有可能会返回599timeout之类，这是一个warning，官方却认为是合理的。

### AsyncHTTPClient 上传图片
AsyncHTTPClient 更高级的用法就是上传图片。例如服务有一个功能就是请求第三方服务的图片OCR服务。需要把用户上传的图片，再转发给第三方服务。
```python
@router.Route('/api/v2/account/upload')
class ApiAccountUploadHandler(helper.BaseHandler):
  @tornado.gen.coroutine
  @helper.token_require
  def post(self, *args, **kwargs):
    upload_type = self.get_argument('type', None)
    files_body = self.request.files['file']
    new_file = 'upload/new_pic.jpg'
    new_file_name = 'new_pic.jpg'

    # 写入文件
    with open(new_file, 'w') as w:
      w.write(file_['body'])
    logging.info('user {} upload {}'.format(user_id, new_file_name))

    # 异步请求 上传图片
    with open(new_file, 'rb') as f:
      files = [('image', new_file_name, f.read())]
    fields = (('api_key', KEY), ('api_secret', SECRET))
    content_type, body = encode_multipart_formdata(fields, files)
    headers = {"Content-Type": content_type, 'content-length': str(len(body))}
    request = tornado.httpclient.HTTPRequest(config.OCR_HOST,
                         method="POST", headers=headers, body=body, validate_cert=False)

    response = yield tornado.httpclient.AsyncHTTPClient().fetch(request)

def encode_multipart_formdata(fields, files):
  """
  fields is a sequence of (name, value) elements for regular form fields.
  files is a sequence of (name, filename, value) elements for data to be
  uploaded as files.
  Return (content_type, body) ready for httplib.HTTP instance
  """
  boundary = '----------ThIs_Is_tHe_bouNdaRY_$'
  crlf = '\r\n'
  l = []
  for (key, value) in fields:
    l.append('--' + boundary)
    l.append('Content-Disposition: form-data; name="%s"' % key)
    l.append('')
    l.append(value)
  for (key, filename, value) in files:
    filename = filename.encode("utf8")
    l.append('--' + boundary)
    l.append(
        'Content-Disposition: form-data; name="%s"; filename="%s"' % (
          key, filename
        )
    )
    l.append('Content-Type: %s' % get_content_type(filename))
    l.append('')
    l.append(value)
  l.append('--' + boundary + '--')
  l.append('')
  body = crlf.join(l)
  content_type = 'multipart/form-data; boundary=%s' % boundary
  return content_type, body


def get_content_type(filename):
  import mimetypes
  return mimetypes.guess_type(filename)[0] or 'application/octet-stream'
```

对比上述的用法，上传图片仅仅是多了一个图片的编码。将图片的二进制数据按照multipart 方式编码。编码的同时，还需要把传递的相关的字段处理好。相比之下，使用requests 的方式则非常简单：
files = {}
f = open('/Users/ghost/Desktop/id.jpg')
files['image'] = f
data = dict(api_key='KEY', api_secret='SECRET')
resp = requests.post(url, data=data, files=files)
f.close()
print resp.status_Code
### 总结
通过AsyncHTTPClient的使用方式，可以轻松的实现handler对第三方服务的请求。结合前面关于tornado异步的使用方式。无非还是两个key。是否需要返回结果，来确定使用callback的方式还是yield的方式。当然，如果不同的函数都yield，yield也可以一直传递。这个特性，tornado的中的tornado.auth 里面对oauth的认证。
大致就是这样的用法。