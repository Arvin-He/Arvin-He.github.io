---
title: Python之jsonrpc
date: 2017-08-25 11:38:39
tags: Python
categories: 编程
---
### 服务端
```python
# server
from http.server import BaseHTTPRequestHandler, HTTPServer
from jsonrpcserver import methods


@methods.add
def ping():
    return('ping')


@methods.add
def hao():
    return('hao')


@methods.add
def hello(name):
    return('Hello, %s' % name)


@methods.add
def Sum(a, b):
    return(a + b)


class TestHttpServer(BaseHTTPRequestHandler):
    def do_POST(self):
        # Process request
        request = self.rfile.read(int(self.headers['Content-Length'])).decode()
        response = methods.dispatch(request)
        # Return response
        self.send_response(response.http_status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(str(response).encode())


if __name__ == '__main__':
    HTTPServer(('localhost', 5000), TestHttpServer).serve_forever()
```

### 客户端
```python
from jsonrpcclient.http_client import HTTPClient

HTTPClient('http://localhost:5000').request('ping')
HTTPClient('http://localhost:5000').request('hao')
HTTPClient('http://localhost:5000').request('hello', 'Nancy')
HTTPClient('http://localhost:5000').request('Sum', 5, 7)
HTTPClient('http://localhost:5000').request('lid')
```

先运行服务端,再运行客户端代码
### 结果
```
--> {"id": 1, "jsonrpc": "2.0", "method": "ping"}
<-- {"jsonrpc": "2.0", "result": "ping", "id": 1} (200 OK)
--> {"id": 2, "jsonrpc": "2.0", "method": "hao"}
<-- {"jsonrpc": "2.0", "result": "hao", "id": 2} (200 OK)
--> {"params": ["Nancy"], "id": 3, "jsonrpc": "2.0", "method": "hello"}
<-- {"jsonrpc": "2.0", "result": "Hello, Nancy", "id": 3} (200 OK)
--> {"params": [5, 7], "id": 4, "jsonrpc": "2.0", "method": "Sum"}
<-- {"jsonrpc": "2.0", "result": 12, "id": 4} (200 OK)
--> {"id": 5, "jsonrpc": "2.0", "method": "lid"}
<-- {"jsonrpc": "2.0", "error": {"code": -32601, "message": "Method not found"}, "id": 5} (404 Not Found)
Traceback (most recent call last):
  File "js_client.py", line 7, in <module>
    HTTPClient('http://localhost:5000').request('lid')
  File "C:\Python34\lib\site-packages\jsonrpcclient\client.py", line 200, in request
    return self.send(Request(method_name, *args, **kwargs))
  File "C:\Python34\lib\site-packages\jsonrpcclient\client.py", line 171, in send
    return self._send_message(request, **kwargs)
  File "C:\Python34\lib\site-packages\jsonrpcclient\http_client.py", line 82, in _send_message
    log_format='<-- %(message)s (%(http_code)s %(http_reason)s)')
  File "C:\Python34\lib\site-packages\jsonrpcclient\client.py", line 114, in _process_response
    response['error'].get('data'))
jsonrpcclient.exceptions.ReceivedErrorResponse: Method not found
```