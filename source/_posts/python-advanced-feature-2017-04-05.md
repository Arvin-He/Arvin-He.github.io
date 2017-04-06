---
title: Python高级特性
date: 2017-04-05 20:06:51
tags: Python
categories: 编程
---
### 1. 列表生成式
```
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# for循环后面还可以加上if判断
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]

# 使用两层循环，生成全排列
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

### 2. 装饰器
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

### 3. 闭包


### 4. 高阶函数


### 5. 偏函数



### 参考文档
* [Python官方文档](https://docs.python.org/3/library/asyncio.html)
* [廖雪峰Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)