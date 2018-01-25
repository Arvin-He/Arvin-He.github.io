---
title: Node最佳实践1
date: 2017-06-06 08:36:55
tags: node
categories: node
---
## 编码规范

### 回调惯例
模块应该公开一个错误优先（error-first）的回调接口。
就像下面这样：
```javascript
module.exports = function (dragonName, callback) { 
     // 这里做一些处理工作  
     var dragon = createDragon(dragonName);
     // 注意, callback第一个参数是 error
     // 这里传入null
     // 如果出错则传入错误信息
     return callback(null, dragon);
}
```

确保在回调中检查错误信息, 要更好地弄明白为什么必须这样做，先想办法创建一个会挂掉的例子，然后修复它。
```javascript
// 这个例子是会挂掉的, 我们很快会修复 :)
var fs = require('fs');
function readJSON(filePath, callback){
     fs.readFile(filePath, function(err, data) {
          callback(JSON.parse(data));
     });
}
readJSON('./package.json', function (err, pkg) { ... }
```

首要问题是 readJSON函数，在执行过程中出现了错误，而这个函数却没有做任何错误检查。你务必要先做错误检查。
改进方案：
```javascript
// 这个例子还是会挂掉 , 很快会修复!
function readJSON(filePath, callback) {
    fs.readFile(filePath, function(err, data) {
     // 这里我们先判断是否有错误发生    
     if (err) {     
          // 出现错误，将错误传入回调函数     
          // 记住: 错误优先（error-first） 回调
          callback(err);    
     }   
     // 如果没有错误则传入null和JSON
     callback(null, JSON.parse(data));
});
}
```

### 将回调函数返回
上面的例子还是存在一个错误，就是如果错误发生了，if  中的表达式不会停止运行，而是继续运行下去。这会导致很多未知的错误。长话短说，务必通过回调函数返回。
```javascript
// 这个例子仍旧会挂掉, 马上修复!
function readJSON(filePath, callback) {
    fs.readFile(filePath, function(err, data) {
    if (err) {
          return callback(err);    
     }    
 
     return callback(null, JSON.parse(data));
  });
}
```

### 仅在同步代码中使用try-catch
几乎完美了！但还有一件事，我们必须要小心 JSON.parse。调用JSON.parse 时，如果传入的字符串无法解析成JSON格式，会抛出异常。
由于JSON.parse是同步发生的，我们可以用try-catch包装起来。**请注意:**你只能对同步代码块做此操作，对回调函数是不起作用的。
```javascript
// 这个例子终于可以正常工作啦 :)
function readJSON(filePath, callback) {
fs.readFile(filePath, function(err, data) {
     var parsedJson;
     // 处理错误
     if (err) {
          return callback(err);
     }
     // 解析JSON
     try {
          parsedJson = JSON.parse(data);
     } catch (exception){
          return callback(exception);
     }
     // 一切工作正常
     return callback(null, parsedJson);
});
}
```

### 尽量避免 this 和 new 关键字
由于Node涉及了大量的回调操作，并且重度使用高阶函数控制流程，因此在Node中绑定一个具体的上下文并不总是行之有效。使用函数式编程风格能避免不少麻烦。
当然，在某些情况下原型（prototype）可能更高效，不过只要可能，还是尽量避免它们。

### 创建微模块
用unix的方式：
> 开发者构建一个程序时应该将其分成很多简单的模块，各部分由定义良好的接口整合,所以问题是局部的，并且能通过替换程序部件的方式在将来的版中加入新特性。
不要创建怪兽般的代码，保持简洁，一个模块就只做一件事，但是要做到极致。

### 使用良好的异步模式

使用async异步处理模块。

### 错误处理
错误可以分为两部分，操作错误和编程错误。

操作错误
精心编写的应用程序中也一样会出现操作错误。因为这些不是 bug ，而是由于操作系统或远程服务导致的，例如：
请求超时, 系统内存不足, 远程连接失败, 

处理操作错误
根据不同运行错误的类型，你可以采用下面的方式处理：
尝试解决错误——如果文件丢失，你可以提前创建一个。
当处理网络通信时，可以重试操作。
把问题告诉客户，表示有些功能不能正常工作——可以用于处理用户输入。
如果错误无法在当前条件下解决，终止进程，例如应用程序无法读取它的配置文件。
还有，上述的所有处理方式都应该记录日志。

编程错误
编程错误都算是bug。下面所列的几条你应该避免，例如：
调用异步函数时没有回调。
不能读取未定义（undefined）的属性

处理编程错误
如果错误属于bug，立刻终止程序，你并不知道应用当前的运行状态。当错误发生时，进程控制系统应该会重启应用程序，例如：supervisord 或者 monit。

## 工作流技巧

### 使用 npm init 创建新项目
init 命令可以帮助你创建应用程序的 package.json 配置文件。文件设置了一些默认配置，之后可以修改。
创建一个优秀的项目应该这样开始：
```
mkdir my-awesome-new-project 
cd my-awesome-new-project 
npm init
```

指定开始和测试脚本。
在你的 package.son 文件中，你可以在 scripts 部分中设置脚本。npm init  默认会创建两个，start 和 test 脚本。可以通过 npm start 和 npm test 命令运行。还有，作为加分项：你可以在这里加入自定义脚本，通过 npm run-script <SCRIPT_NAME> 来运行。
注意，NPM 会通过设置 $PATH 来扫描 node_modules/.bin 下的所有可执行脚本。这样可以避免安装全局的 NPM 模块。

### 环境变量
生产部署和演示部署都应该由环境变量来实现。最主流的实现方式是同时在生产和演示中设置 NODE_ENV变量。
根据你设置的环境变量，你可以使用 nconf 模块来加载配置信息。
当然，你也可以在你的Node.js 应用中使用其它环境变量设置 process.env，这是一个包含了用户环境的对象。

### 不要重新发明轮子
务必优先寻找现成的解决方案。NPM 的库超级多，涵盖了你平时需要的大部分功能。

### 使用风格指南
所有的代码都保持统一风格有助于理解大型代码库。其中应该包含缩进、变量命名、最佳实践以及其他方面。
如果想看一个实际的例子，请查看  RisingStack 编写的 [Node.js 风格指南](https://github.com/RisingStack/node-style-guide)。

### 保持风格一致
JSCS 是一个JavaScript 编码风格检查工具。将JSCS加入项目对你来说小菜一碟：`npm install jscs --save-dev`, 你需要做的下一步关键就是在 package.json 文件中加入下面的代码来开启它：
```
scripts: {  
    "jscs": "jscs index.js"
}
```

当然，你也可以加入多个文件、目录检查。但为什么我们仅仅在 package.json 文件中创建了一个自定义的脚本呢？我们是以本地的方式安装 jscs 的，所以在一个系统中可以有多个不同版本。这样还能正常工作是因为NPM 执行时会将 `node_modules/.bin`  设置到 PATH上。

你可以在`.jscsrc` 文件中定义验证规则，或者使用预设规则。从这里可以查看可用的预设，通过 `--preset=[PRESET_NAME]`来应用。

执行 JSHint、JSCS 规则
你的构建过程还应该包含 JSHint 和 JSCS，不过在开发者的电脑上运行 `pre-commit checks` 或许是个不错的主意。
要实现这个很简单，你可以使用 pre-commit NPM 库：`npm install --save-dev pre-commit`, 然后在 package.json 文件中作如下配置：
```
pre-commit": [  
    "jshint",
    "jscs"
],
```

注意，pre-commit 将会扫描 package.json中script里的所有脚本。开启以后，每次提交时都会自动进行检查。

### 用JS替换JSON做配置
我们看到大量的项目都是使用JSON文件做配置的。这是目前最普遍的做法，JS配置文件则能够提供更大的灵活度。所以我们推荐你使用 config.js 文件：
```javascript
var url =require('url');
var config = module.exports = {};
var redisToGoConfig;
 
config.server = {
  host: '0.0.0.0',
  port: process.env.PORT || 3000
};
 
// look, a comment in the config file!
// would be tricky in a JSON ;)
config.redis = {
  host: 'localhost',
  port: 6379,
  options: {
 
}
};
 
if (process.env.REDISTOGO_URL) {
  redisToGoConfig = url.parse(process.env.REDISTOGO_URL);
  config.redis.port = redisToGoConfig.port;
  config.redis.host = redisToGoConfig.hostname;
  config.redis.options.auth_pass = redisToGoConfig.auth.split(':')[1];
}
```

### 使用 NODE_PATH
你是否曾经碰到过下面这种情况？
```javascript
var myModule = require('../../../../lib/myModule');
 
myModule.doSomething(function (err) {

});
```

当你的项目结构变得错综复杂，模块依赖会非常麻烦。要解决这个问题有两个办法：

1. 把你的模块软链接到node_modules目录下。
2. 使用 NODE_PATH。
在RisingStack我们使用 `NODE_PATH` 的方式，因为将所有相关文件软链接到 node_modules目录需要大量额外的工作，并且在很多操作系统下都不适用。

设置 NODE_PATH
假设你的项目结构是这样的：
```
--lib
  --model
    --car.js
  --index.js
  --package.json
```

我们可以使用 指向 lib 目录的`NODE_PATH`，而不是使用相对路径。在我们的package.json 的 `start script`部分，我们使用`NODE_PATH`设置并且用`npm start `运行项目。
```javascript
var Car = require('model/Car');
 
 console.log('I am a Car!');
 
{
  "name": "node_path",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "NODE_PATH=lib node index.js"
},
  "author": "",
  "license": "ISC"
}
```

### 依赖注入
依赖注入是一种软件设计模式，是指将一到多个依赖（或服务）注入或通过引用的方式引入到需要依赖的对象。 依赖注入在测试中非常有用。使用这个模式你可以轻松模拟模块间的依赖关系。
```javascript
function userModel (options) {
var db;
if (!options.db) {
  throw new Error('Options.db is required');
}
db = options.db;
return {
  create: function (done) {
    db.query('INSERT ...', done);
  }
}
}
 
module.exports = userModel;
```

```javascript
var db = require('db');
// do some init here, or connect
db.init();
 
var userModel = require('User')({
db: db
});
 
userModel.create(function (err, user) {
});
```

```javascript
var test = require('tape');
var userModel = require('User');
 
test('it creates a user with id', function (t) {
var user = {
id: 1
};
var fakeDb = {
query: function (done) {
done(null, user);
}
}
userModel({
db: fakeDb
}).create(function (err, user) {
t.equal(user.id, 1, 'User id should match');
t.end();
})
});
```

上面的例子中我们有两个不同的 db。在 index.js 文件中是“真实的” db 模块，而第二段代码中我们只是简单地创建了一个模拟的db模块。

这样我们在测试时就可以轻松地将模拟的依赖引入模块。








### 参考
* [NodeJS 最佳实践](http://web.jobbole.com/84401/)