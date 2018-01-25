---
title: Node笔记1
date: 2017-06-01 15:45:03
tags: node
categories: node
---
### 创建一个http服务器
```javascript
// 第一行请求（require）Node.js自带的 http 模块，并且把它赋值给 http 变量。
var http = require("http");
// 调用http模块提供的函数:createServer.该函数会返回一个对象
// 该对象有一个listen的方法,该方法有一个数值参数,指定这个HTTP服务器监听的端口号。
http.createServer(function(request, response)
{
    response.writeHead(200, {"Content-Type":"text/plain"});
    response.write("Hello World");
    response.end();
}).listen(8888);
```
在命令行输入:`node server.js`,打开浏览器访问http://localhost:8888/，你会看到一个写着“Hello World”的网页。
注意:createServer 函数的参数是一个匿名函数。即`function(request, response){...}`.

### 基于事件驱动的回调
当使用 http.createServer 方法的时候，不只想要侦听某个端口，还想要它在服务器收到一个HTTP请求的时候做点什么。但问题是，这是异步的：请求任何时候都可能到达，但是服务器却跑在一个单进程中。那么Node.js程序中，当一个新的请求到达8888端口的时候，我们怎么控制流程呢？
嗯，这就需要Node.js/JavaScript的事件驱动,现在看看这些概念是怎么应用在我们的服务器代码里的。
我们创建了服务器，并且向创建它的方法传递了一个函数。无论何时服务器收到一个请求，这个函数就会被调用。我们不知道这件事情什么时候会发生，但是我们现在有了一个处理请求的地方：它就是我们传递过去的那个匿名函数。至于它是被预先定义的函数还是匿名函数，就无关紧要了。这个就是传说中的 回调 。我们给某个方法传递了一个函数，这个方法在有相应事件发生时调用这个函数来进行回调 。

```javascript
var http = require("http");

// http.createServer(function(request, response)
// {
//     response.writeHead(200, {"Content-Type":"text/plain"});
//     response.write("Hello World");
//     response.end();
// }).listen(8888);

function onRequest(request, response)
{
    console.log("Request receives.");
    response.writeHead(200, {"Content-Type":"text/plain"});
    response.write("Hello World");
    response.end();
}

http.createServer(onRequest).listen(8888);

console.log("Server has started.");
```

当运行`node server.js`时，它会马上在命令行上输出“Server has started.”。当我们向服务器发出请求（在浏览器访问http://localhost:8888/ ）或者强制刷新页面时，“Request received.”这条消息就会在命令行中出现。这就是事件驱动的异步服务器端JavaScript和它的回调！
**注意:**当我们在服务器访问网页时，服务器可能会输出两次“Request received.”。那是因为大部分浏览器都会在你访问 http://localhost:8888/ 时尝试读取http://localhost:8888/favicon.ico )

### 服务器是如何处理请求的
当回调启动，即onRequest()函数被触发的时候,有两个参数被传入:request 和 response.
它们是对象，你可以使用它们的方法来处理HTTP请求的细节，并且响应请求（比如向发出请求的浏览器发回一些东西）。所以当收到请求时，使用 response.writeHead() 函数发送一个HTTP状态200和HTTP头的内容类型（content-type），使用 response.write() 函数在HTTP相应主体中发送文本“Hello World"。最后，调用 response.end() 完成响应。目前来说，我们对请求的细节并不在意，所以我们没有使用 request 对象。

### 关于变量命名
Node.js中自带了一个叫做“http”的模块，在代码中请求它并把返回值赋给一个本地变量http。这样本地变量http就变成了一个拥有所有 http 模块所提供的公共方法的对象。给这种本地变量起一个和模块名称一样的名字是一种惯例，但你也可以按照自己的喜好来.

### 创建自己的模块
我们把 server.js 变成一个真正的模块，你就能搞明白了。
事实上，我们不用做太多的修改。把某段代码变成模块意味着需要把我们希望提供其功能的部分 导出到请求这个模块的脚本。目前，HTTP服务器需要导出的功能非常简单，因为请求服务器模块的脚本仅仅是需要启动服务器而已。我们把服务器脚本放到一个叫做 start 的函数里，然后我们导出这个函数。
```javascript
// server.js
var http = require("http");

function start() {
  function onRequest(request, response) {
    console.log("Request received.");
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}
// 导出start函数
exports.start = start;
```
这样，现在就可以创建我们的主文件 index.js 并在其中启动我们的HTTP了，虽然服务器的代码还在 server.js 中。创建 index.js 文件并写入以下内容：
```javascript
var server = require("./server");
server.start();
```
正如上面所示代码，我们可以像使用任何其他的内置模块一样使用server模块：请求这个server.js文件并把它指向一个server变量，导出的start函数就可以被我们使用了。
好了。现在可以从我们的主要脚本启动我们的的应用了，在命令行输入：`node index.js`

### 路由
处理不同的HTTP请求在代码中是一个不同的部分，叫做“路由选择”，接下来就创造一个叫做"路由"的模块。

我们要为路由提供请求的URL和其他需要的GET及POST参数，随后路由需要根据这些数据来执行相应的代码（这里“代码”对应整个应用的第三部分：一系列在接收到请求时真正工作的处理程序）。

因此，我们需要查看HTTP请求，从中提取出请求的URL以及GET/POST参数。这一功能应当属于路由还是服务器（甚至作为一个模块自身的功能）确实值得探讨，但这里暂定其为我们的HTTP服务器的功能。

我们需要的所有数据都会包含在request对象中，该对象作为onRequest()回调函数的第一个参数传递。但是为了解析这些数据，需要额外的Node.JS模块，它们分别是url和querystring模块。当然我们也可以用querystring模块来解析POST请求体中的参数.现在我们来给onRequest()函数加上一些逻辑，用来找出浏览器请求的URL路径：

```javascript
// server.js
var http = require("http");
var url = require("url");

function start() {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```

现在可以根据请求的URL路径来区别不同请求了,然后使用路由将请求以URL路径为基准映射到处理程序上。现在以来编写路由了，建立一个名为router.js的文件，添加以下内容：
```javascript
function route(pathname) {
  console.log("About to route a request for " + pathname);
}

exports.route = route;
```

如你所见，这段代码什么也没干，不过对于现在来说这是应该的。在添加更多的逻辑之前，我们需要先来看看如何把路由和服务器整合起来。使用硬编码的方式将这一依赖项绑定到服务器上是一件非常痛苦的事，因此我们采用依赖注入的方式来添加路由模块,这种方式较为松散.

首先，来扩展一下服务器的start()函数，以便将路由函数作为参数传递过去:
```javascript
var http = require("http");
var url = require("url");

// 注意:start函数传进一个route参数
function start(route) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    // 这个route函数就是router模块里的route函数
    route(pathname);

    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```

同时，相应扩展index.js，使得路由函数可以被注入到服务器中：
```javascript
var server = require("./server");
var router = require("./router");

// 这里将router模块的route函数传进start函数中去
server.start(router.route);
```

在这里，传递的route函数依旧什么也没做。如果现在启动应用随后请求一个URL，你将会看到应用输出相应的信息，这表明我们的HTTP服务器已经在使用路由模块了，并会将请求的路径传递给路由.输出结果为:
```
C:\Users\aron\AppData\Roaming\aronWorkSpace\arvinGithub\node>node index.js
Server has started.
Request for / received.
About to route a request for /
Request for /favicon.ico received.
About to route a request for /favicon.ico
```

### 参考
* [Node入门](http://www.nodebeginner.org/index-zh-cn.html#execution-in-the-kongdom-of-verbs)
