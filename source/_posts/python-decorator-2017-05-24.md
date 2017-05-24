---
title: Python之装饰器
date: 2017-05-24 13:31:32
tags: Python
categories: 编程
---
### 装饰器概念
谈装饰器前，还要先要明白一件事，Python 中的函数和 Java、C++不太一样，Python 中的函数可以像普通变量一样当做参数传递给另外一个函数.
装饰器本质上是一个 Python 函数或类，其返回值也是一个函数或类对象, 作用是让其他函数或类在不需要做任何代码修改的前提下增加额外功能。
```python
#!/usr/bin/env
# -*-coding:utf-8-*-
__author__ = 'arvin'
def decorator(func):
    print("%s was called" % func.__name__)
    func()
def hello(name="arvin"):
    print("Hello %s!" % name)
decorator(hello)
```

### 装饰器应用场景
装饰器经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景，装饰器是解决这类问题的绝佳设计。
有了装饰器，就可以抽离出大量与函数功能本身无关的雷同代码到装饰器中并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。

### @ 语法糖
@ 符号其实就是装饰器的语法糖，它放在函数开始定义的地方，这样就可以省略最后一步再次赋值的操作。
```python
def log(func):
    def wrapper():
        logging.warn("%s is running" % func.__name__)
        return func()
    return wrapper

@log
def foo():
    print("this is foo function")

foo()
```
有了 @ ，就可以省去foo = log(foo)这一句了，直接调用 foo() 即可得到想要的结果。foo() 函数不需要做任何修改，只需在定义的地方加上装饰器，调用的时候还是和以前一样，如果有其他的类似函数，可以继续调用装饰器来修饰函数，而不用重复修改函数或者增加新的封装。这样，就提高了程序的可重复利用性，并增加了程序的可读性。

装饰器在 Python 使用如此方便都要归因于 Python 的函数能像普通的对象一样能作为参数传递给其他函数，可以被赋值给其他变量，可以作为返回值，可以被定义在另外一个函数内。

### 不带参数的装饰器
在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）,本质上，decorator就是一个返回函数的高阶函数.

定义装饰器
```python
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```
其中,wrapper名字是自定义的,上面的log是一个decorator,接受一个函数作为参数，并返回一个函数。

使用装饰器
```python
@log
def now():
    print('2017-4-25')
```
调用now()函数，不仅会运行now()函数本身，还会在运行now()函数前打印一行日志.

### 带参数的装饰器
装饰器的语法允许我们在调用时，提供其它参数，比如@decorator(a)。这样，就为装饰器的编写和使用提供了更大的灵活性。
比如，我们可以在装饰器中指定日志的等级.
```python
def use_logging(level):
    def decorator(func):
        def wrapper(*args, **kwargs):
            if level == "warn":
                logging.warn("%s is running" % func.__name__)
            elif level == "info":
                logging.info("%s is running" % func.__name__)
            return func(*args)
        return wrapper
    return decorator

@use_logging(level="warn")
def foo(name='foo'):
    print("this is %s" % name)
```
上面的 use_logging 是允许带参数的装饰器。它实际上是对原有装饰器的一个函数封装，并返回一个装饰器。我们可以将它理解为一个含有参数的闭包。当我
们使用@use_logging(level="warn")调用的时候，Python 能够发现这一层的封装，并把参数传递到装饰器的环境中。
@use_logging(level="warn")等价于@decorator

### 类装饰器
相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。使用类装饰器主要依靠类的\_\_call\_\_方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。
```python
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print ('class decorator runing')
        self._func()
        print ('class decorator ending')

@Foo
def bar():
    print ('bar')

bar()
```
### 使用functools.wraps
使用装饰器极大地复用了代码，但有一个缺点就是原函数的元信息不见了，比如函数的docstring、\_\_name\_\_、参数列表，先看例子：
```python
# 装饰器
def logged(func):
    def with_logging(*args, **kwargs):
        print func.__name__      # 输出 'with_logging'
        print func.__doc__       # 输出 None
        return func(*args, **kwargs)
    return with_logging

# 函数
@logged
def f(x):
   """does some math"""
   return x + x * x

logged(f)
```
不难发现，函数 f 被with\_logging取代了，当然它的docstring，\_\_name\_\_就是变成了with_logging函数的信息了。
好在有functools.wraps，wraps本身也是一个装饰器，它能把原函数的元信息拷贝到装饰器里面的 func 函数中，
这使得装饰器里面的 func 函数也有和原函数 foo 一样的元信息了。
```python
from functools import wraps
def logged(func):
    @wraps(func)
    def with_logging(*args, **kwargs):
        print func.__name__      # 输出 'f'
        print func.__doc__       # 输出 'does some math'
        return func(*args, **kwargs)
    return with_logging

@logged
def f(x):
   """does some math"""
   return x + x * x
```
### 装饰器顺序
一个函数还可以同时定义多个装饰器，比如:
```python
@a
@b
@c
def f ():
    pass
```
它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于`f = a(b(c(f)))`

### 在函数执行之后进行装饰
```python
#!/usr/bin/env
# -*-coding:utf-8-*-
__author__ = 'arvin'

def decorator(func):
    def wrapper():
      print("%s was called" % func.__name__)
      func()
      print("bye~")
    return wrapper

@decorator
def hello(name="arvin"):
    print("Hello %s!" % name)

hello()

# 执行
outputs:
hello was called
Hello arvin!
bye~
```
可以这样理解hello()==decorator(hello)()==wrapper()，最后其实就是执行wrapper()函数而已

```python
#!/usr/bin/env
# -*-coding:utf-8-*-
__author__ = 'arvin'
from functools import wraps
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("%s was called" % func.__name__)
        func(*args, **kwargs)
        print("bye~")
    return wrapper

@decorator
def hello(name="arvin"):
    print("Hello %s!" % name)
hello('world')
```

### 参考
* [https://juejin.im/post/587c565f128fe1006b00c973](https://juejin.im/post/587c565f128fe1006b00c973)
* [http://www.jianshu.com/p/ada1ceecd079](http://www.jianshu.com/p/ada1ceecd079)