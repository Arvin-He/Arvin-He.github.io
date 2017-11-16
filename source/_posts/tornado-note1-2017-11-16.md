---
title: tornado学习笔记(一)
date: 2017-11-16 19:42:00
tags: tornado
categories: python
---
### 说明
高性能源于Tornado基于Epoll（unix为kqueue）的异步网络IO。因为tornado的单线程机制，一不小心就容易写出阻塞服务（block）的代码。不但没有性能提高，反而会让性能急剧下降。因此，探索tornado的异步使用方式很有必要。

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
```
使用ab测试一下：
ab -c 5 -n 5 http://127.0.0.1:5000/sync

Server Software:    TornadoServer/4.3
Server Hostname:    127.0.0.1
Server Port:      5000

Document Path:     /sync
Document Length:    5 bytes

Concurrency Level:   5
Time taken for tests:  5.076 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   985 bytes
HTML transferred:    25 bytes
Requests per second:  0.99 [#/sec] (mean)
Time per request:    5076.015 [ms] (mean)
Time per request:    1015.203 [ms] (mean, across all concurrent requests)
Transfer rate:     0.19 [Kbytes/sec] received
仅有可怜的 0.99，姑且当成每秒处理一个请求吧。
```

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
### tornado基本使用
以下是一个简单的例子,为简单分析,去除了一些无关紧要的参数和代码,不要注意代码本身,注重tornado使用流程.

```python
# -*- coding: utf-8 -*-
import json
import requests
import tornado.gen
import tornado.web
import tornado.ioloop
import tornado.httpserver

class MainHandler(tornado.web.RequestHandler):
    def initialize(self):
        pass

    def post(self):
        pass

    @tornado.web.asynchronous
    @tornado.gen.coroutine
    def get(self):
        try:
            company_info = yield self.get_company_info()
            if isinstance(company_info, list):
                self.write(json.dumps(company_info))
            else:
                self.write(json.dumps({'result': company_info}))
            self.finish()
        except Exception as e:
            print(e)

    @tornado.gen.coroutine
    def get_company_info(self):
        request_data = self.get_request_data()
        if isinstance(request_data, dict):
            response_data = self.get_response_data(request_data)
            if response_data != '验证码错误':
                return response_data
            else:
                return '验证码错误'
        else:
            return request_data

    def get_request_data(self):
        request_data = self.parse_params.get_request_params()
        if request_data:
            if 'url' in request_data.keys():
                request_data.pop('url')
            request_data['__EVENTTARGET'] = ''
            request_data['__EVENTARGUMENT'] = ''
            company = self.get_argument('company', None)
            if company:
                request_data['txtCompanyName'] = company
                if self.check_request_params(request_data):
                    return request_data
                else:
                    return '内部请求参数错误'
            else:
                return '请求参数<企业名称>为空'
        else:
            return '内部请求参数异常'

    def get_response_data(self, request_data):
        try:
            if self.parse_params.use_proxy:
                response = requests.post(self.url, data=request_data, headers=self.headers, proxies=self.parse_params.proxy)
            else:
                response = requests.post(self.url, data=request_data, headers=self.headers)
            response_data = self.parse_company_info(response)
            return response_data
        except requests.exceptions.RequestException as e:
            return '返回页面异常'

    def parse_company_info(self, html):
        response_data = []
        if html:
            soup = BeautifulSoup(html.text, 'lxml')
            for script_tag in soup.find_all('script'):
                if '验证码错误' in str(script_tag.string):
                    logger.warning('验证码错误')
                    return '验证码错误'
            for td_tag in soup.find_all('td'):
                if '未查询到匹配结果' in str(td_tag.string):
                    return '未查询到匹配结果'
            for table_tag in soup.find_all('table'):
                if table_tag.get('id') == 'gvCmpany':
                    company_detail = list()
                    td_list = table_tag.find_all('td')
                    td_list = [str(item.string).strip() for item in td_list]
                    for i in range(0, len(td_list), len(self.table_head)):
                        company_detail.append(td_list[i:i + len(self.table_head)])
                    if company_detail:
                        for item_list in company_detail:
                            response_data.append(dict(zip(self.table_head, item_list)))
                    return response_data
        else:
            return '返回页面为空页'


if __name__ == '__main__':
    port = 9630
    try:
        app = tornado.web.Application(
            [(r'/', MainHandler)], autoreload=True)
        if sys.platform == 'win32':
            app.listen(port)
        else:
            server = tornado.httpserver.HTTPServer(app)
            server.bind(port)
            # forks one process per cpu
            server.start()
        tornado.ioloop.IOLoop.current().start()
    except Exception as e:
        logger.error(e)
```

tornado虽然以异步非阻塞高性能著称的框架,但默认不是异步非阻塞的,你需要添加两个装饰器@tornado.web.asynchronous和@tornado.gen.coroutine,
这样才算是异步非阻塞的.然而当你用tornado作为http服务器,同时用了requests库发送请求后,你会发现请求还是一个一个的处理,根本不是异步非阻塞的.
原因是requests库发送请求是阻塞的,它会阻塞整个tornado服务进程,为什么阻塞,原因是requests


尽管在执行异步任务的时候选择了timeout 1秒，主线程的返回还是很快的。ab压测如下：
Document Path:     /async
Document Length:    5 bytes

Concurrency Level:   5
Time taken for tests:  0.009 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   985 bytes
HTML transferred:    25 bytes
Requests per second:  556.92 [#/sec] (mean)
Time per request:    8.978 [ms] (mean)
Time per request:    1.796 [ms] (mean, across all concurrent requests)
Transfer rate:     107.14 [Kbytes/sec] received

上述的使用方式，通过tornado的IO循环，把可以把耗时的任务放到后台异步计算，请求可以接着做别的计算。可是，经常有一些耗时的任务完成之后，我们需要其计算的结果。此时这种方式就不行了。车道山前必有路，只需要切换一异步方式即可。下面使用协程来改写：
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

可以看到异步在处理，而结果值也被返回了。
Server Software:    TornadoServer/4.3
Server Hostname:    127.0.0.1
Server Port:      5000

Document Path:     /async/task
Document Length:    5 bytes

Concurrency Level:   5
Time taken for tests:  0.049 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   985 bytes
HTML transferred:    25 bytes
Requests per second:  101.39 [#/sec] (mean)
Time per request:    49.314 [ms] (mean)
Time per request:    9.863 [ms] (mean, across all concurrent requests)
Transfer rate:     19.51 [Kbytes/sec] received

qps提升还是很明显的。有时候这种协程处理，未必就比同步快。在并发量很小的情况下，IO本身拉开的差距并不大。甚至协程和同步性能差不多。例如你跟博尔特跑100米肯定输给他，可是如果跟他跑2米，鹿死谁手还未定呢。
yield挂起函数协程，尽管没有block主线程，因为需要处理返回值，挂起到响应执行还是有时间等待，相对于单个请求而言。另外一种使用异步和协程的方式就是在主线程之外，使用线程池，线程池依赖于futures。Python2需要额外安装。
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

再运行ab测试：
Document Path:     /future
Document Length:    5 bytes

Concurrency Level:   5
Time taken for tests:  0.003 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   995 bytes
HTML transferred:    25 bytes
Requests per second:  1912.78 [#/sec] (mean)
Time per request:    2.614 [ms] (mean)
Time per request:    0.523 [ms] (mean, across all concurrent requests)
Transfer rate:     371.72 [Kbytes/sec] received

qps瞬间达到了1912.78。同时，可以看到服务器的log还在不停的输出ping的结果。
想要返回值也很容易。再切换一下使用方式接口。使用tornado的gen模块下的with_timeout功能（这个功能必须在tornado>3.2的版本）。
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

线程池的方式也可以通过使用tornado的yield把函数挂起，实现了协程处理。可以得出耗时任务的result，同时不会block住主线程。
Concurrency Level:   5
Time taken for tests:  0.043 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   960 bytes
HTML transferred:    0 bytes
Requests per second:  116.38 [#/sec] (mean)
Time per request:    42.961 [ms] (mean)
Time per request:    8.592 [ms] (mean, across all concurrent requests)
Transfer rate:     21.82 [Kbytes/sec] received
qps为116，使用yield协程的方式，仅为非reponse的十分之一左右。看起来性能损失了很多，主要原因这个协程返回结果需要等执行完毕任务。
好比打鱼，前一种方式是撒网，然后就完事，不闻不问，时间当然快，后一种方式则撒网之后，还得收网，等待收网也是一段时间。当然，相比同步的方式还是快了千百倍，毕竟撒网还是比一只只钓比较快。
具体使用何种方式，更多的依赖业务，不需要返回值的往往需要处理callback，回调太多容易晕菜，当然如果需要很多回调嵌套，首先优化的应该是业务或产品逻辑。yield的方式很优雅，写法可以异步逻辑同步写，爽是爽了，当然也会损失一定的性能。
异步多样化
Tornado异步服务的处理大抵如此。现在异步处理的框架和库也很多，借助redis或者celery等，也可以把tonrado中一些业务异步化，放到后台执行。
此外，Tornado还有客户端异步功能。该特性主要是在于 AsyncHTTPClient的使用。此时的应用场景往往是tornado服务内，需要针对另外的IO进行请求和处理。顺便提及，上述的例子中，调用ping其实也算是一种服务内的IO处理。接下来，将会探索一下AsyncHTTPClient的使用，尤其是使用AsyncHTTPClient上传文件与转发请求。
异步客户端
前面了解Tornado的异步任务的常用做法，姑且归结为异步服务。通常在我们的服务内，还需要异步的请求第三方服务。针对HTTP请求，Python的库Requests是最好用的库，没有之一。官网宣称：HTTP for Human。然而，在tornado中直接使用requests将会是一场恶梦。requests的请求会block整个服务进程。
上帝关上门的时候，往往回打开一扇窗。Tornado提供了一个基于框架本身的异步HTTP客户端（当然也有同步的客户端）--- AsyncHTTPClient。
AsyncHTTPClient 基本用法
AsyncHTTPClient是 tornado.httpclinet 提供的一个异步http客户端。使用也比较简单。与服务进程一样，AsyncHTTPClient也可以callback和yield两种使用方式。前者不会返回结果，后者则会返回response。
如果请求第三方服务是同步方式，同样会杀死性能。
class SyncHandler(tornado.web.RequestHandler):
  def get(self, *args, **kwargs):

    url = 'https://api.github.com/'
    resp = requests.get(url)
    print resp.status_code

    self.finish('It works')

使用ab测试大概如下：
Document Path:     /sync
Document Length:    5 bytes

Concurrency Level:   5
Time taken for tests:  10.255 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   985 bytes
HTML transferred:    25 bytes
Requests per second:  0.49 [#/sec] (mean)
Time per request:    10255.051 [ms] (mean)
Time per request:    2051.010 [ms] (mean, across all concurrent requests)
Transfer rate:     0.09 [Kbytes/sec] received

性能相当慢了，换成AsyncHTTPClient再测：
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

qps 提高了很多
Document Path:     /async
Document Length:    5 bytes

Concurrency Level:   5
Time taken for tests:  0.162 seconds
Complete requests:   5
Failed requests:    0
Total transferred:   985 bytes
HTML transferred:    25 bytes
Requests per second:  30.92 [#/sec] (mean)
Time per request:    161.714 [ms] (mean)
Time per request:    32.343 [ms] (mean, across all concurrent requests)
Transfer rate:     5.95 [Kbytes/sec] received

同样，为了获取response的结果，只需要yield函数。
class AsyncResponseHandler(tornado.web.RequestHandler):
  @tornado.web.asynchronous
  @tornado.gen.coroutine
  def get(self, *args, **kwargs):

    url = 'https://api.github.com/'
    http_client = tornado.httpclient.AsyncHTTPClient()
    response = yield tornado.gen.Task(http_client.fetch, url)
    print response.code
    print response.body

AsyncHTTPClient 转发
使用Tornado经常需要做一些转发服务，需要借助AsyncHTTPClient。既然是转发，就不可能只有get方法，post，put，delete等方法也会有。此时涉及到一些 headers和body，甚至还有https的waring。
下面请看一个post的例子， yield结果，通常，使用yield的时候，handler是需要 tornado.gen.coroutine。
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

AsyncHTTPClient 构造请求
如果业务处理并不是在handlers写的，而是在别的地方，当无法直接使用tornado.gen.coroutine的时候，可以构造请求，使用callback的方式。
body = urllib.urlencode(params)
req = tornado.httpclient.HTTPRequest(
 url=url, 
 method='POST', 
 body=body, 
 validate_cert=False) 

http_client.fetch(req, self.handler_response)

def handler_response(self, response):

  print response.code

用法也比较简单，AsyncHTTPClient中的fetch方法，第一个参数其实是一个HTTPRequest实例对象，因此对于一些和http请求有关的参数，例如method和body，可以使用HTTPRequest先构造一个请求，再扔给fetch方法。通常在转发服务的时候，如果开起了validate_cert，有可能会返回599timeout之类，这是一个warning，官方却认为是合理的。
AsyncHTTPClient 上传图片
AsyncHTTPClient 更高级的用法就是上传图片。例如服务有一个功能就是请求第三方服务的图片OCR服务。需要把用户上传的图片，再转发给第三方服务。
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

对比上述的用法，上传图片仅仅是多了一个图片的编码。将图片的二进制数据按照multipart 方式编码。编码的同时，还需要把传递的相关的字段处理好。相比之下，使用requests 的方式则非常简单：
files = {}
f = open('/Users/ghost/Desktop/id.jpg')
files['image'] = f
data = dict(api_key='KEY', api_secret='SECRET')
resp = requests.post(url, data=data, files=files)
f.close()
print resp.status_Code
总结
通过AsyncHTTPClient的使用方式，可以轻松的实现handler对第三方服务的请求。结合前面关于tornado异步的使用方式。无非还是两个key。是否需要返回结果，来确定使用callback的方式还是yield的方式。当然，如果不同的函数都yield，yield也可以一直传递。这个特性，tornado的中的tornado.auth 里面对oauth的认证。
大致就是这样的用法。