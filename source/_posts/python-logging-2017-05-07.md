---
title: Python之logging模块
date: 2017-05-07 15:46:55
tags: Python
categories: 编程
---
### 日志设定
在软件开发过程中,需要将日志信息输出到控制台并写入到日志文件中.但是如何做呢?

```python
import logging  

# 创建一个logger  
logger = logging.getLogger(__file__)  
logger.setLevel(logging.DEBUG)  
   
# 创建一个handler，用于写入日志文件  
fh = logging.FileHandler('mylog.log')  
fh.setLevel(logging.DEBUG)  
   
# 再创建一个handler，用于输出到控制台  
ch = logging.StreamHandler()  
ch.setLevel(logging.DEBUG)  
   
# 定义handler的输出格式  
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')  
fh.setFormatter(formatter)  
ch.setFormatter(formatter)  
  
# 给logger添加handler  
logger.addHandler(fh)  
logger.addHandler(ch)  
  
# 记录一条日志  
logger.info('this is first log.')  
```

### logging模块的API
`logging.getLogger([name])`
返回一个logger实例，如果没有指定name，返回root logger。只要name相同，返回的logger实例都是同一个而且只有一个，即name和logger实例是一一对应的。这意味着，无需把logger实例在各个模块中传递。只要知道name，就能得到同一个logger实例.

`Logger.setLevel(lvl)`
设置logger的level， level有以下几个级别：NOTSET < DEBUG < INFO < WARNING < ERROR < CRITICAL

`Logger.addHandler(hdlr)`
logger可以雇佣handler来帮它处理日志， handler主要有以下几种：StreamHandler: 输出到控制台FileHandler:   输出到文件handler还可以设置自己的level以及输出格式。

`logging.basicConfig([**kwargs])`
这个函数用来配置root logger， 为root logger创建一个StreamHandler，设置默认的格式。
这些函数： logging.debug()、logging.info()、logging.warning()、   logging.error()、logging.critical() 如果调用的时候发现root logger没有任何   handler， 会自动调用basicConfig添加一个handler* 如果root logger已有handler， 这个函数不做任何事情
使用basicConfig来配置root logger的输出格式和level：

```python
import logging  
logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)  
logging.debug('This message should appear on the console')  
```

### 关于root logger以及logger的父子关系
关于root logger， 实际上logger实例之间还有父子关系， root logger就是处于最顶层的logger， 它是所有logger的祖先。如下图:

![logger实例之间关系](python-logging-2017-05-07)

root logger是默认的logger
如果不创建logger实例， 直接调用logging.debug()、logging.info()logging.warning()、logging.error()、logging.critical()这些函数，那么使用的logger就是 root logger， 它可以自动创建，也是单实例的。

如何得到root logger
通过logging.getLogger()或者logging.getLogger("")得到root logger实例。

默认的levelroot 
logger默认的level是logging.WARNING

如何表示父子关系
logger的name的命名方式可以表示logger之间的父子关系. 比如：
```python
parent_logger = logging.getLogger('foo')
child_logger = logging.getLogger('foo.bar')
```

什么是effective level
logger有一个概念，叫effective level。 如果一个logger没有显示地设置level，那么它就用父亲的level。如果父亲也没有显示地设置level， 就用父亲的父亲的level，以此推....最后到达root logger，一定设置过level。默认为logging.WARNING
child loggers得到消息后，既把消息分发给它的handler处理，也会传递给所有祖先logger处理，

```python

import logging  
   
# 设置root logger  
r = logging.getLogger()  
ch = logging.StreamHandler()  
ch.setLevel(logging.DEBUG)  
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')  
ch.setFormatter(formatter)  
r.addHandler(ch)  
   
# 创建一个logger作为父亲  
p = logging.getLogger('foo')  
p.setLevel(logging.DEBUG)  
ch = logging.StreamHandler()  
ch.setLevel(logging.DEBUG)  
formatter = logging.Formatter('%(asctime)s - %(message)s')  
ch.setFormatter(formatter)  
p.addHandler(ch)  
   
# 创建一个孩子logger  
c = logging.getLogger('foo.bar')  
c.debug('foo') 
```

```
2017-05-07 16:04:29,893 - foo  
2011-05-07 16:04:29,893 - DEBUG - foo 
```
可见， 孩子logger没有任何handler，所以对消息不做处理。但是它把消息转发给了它的父亲以及root logger。最后输出两条日志。

### 参考
* [使用python的logging模块](http://kenby.iteye.com/blog/1162698)
