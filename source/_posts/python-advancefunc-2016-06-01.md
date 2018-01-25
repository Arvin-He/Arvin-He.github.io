---
title: Python之高阶函数
date: 2017-06-01 11:39:39
tags: python
categories: python
---
### 高阶函数定义
高阶函数英文叫 Higher-order function, 一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

### 变量可以指向函数
```python
In [18]: abs(-5)         
Out[18]: 5               
                         
In [19]: abs             
Out[19]: <function abs>  
                         
In [20]: x = abs(-10)    
                         
In [21]: x               
Out[21]: 10              
                         
In [22]: f = abs         
                         
In [23]: f               
Out[23]: <function abs>  

In [24]: f(-6)
Out[24]: 6
```
由上面的示例得出：函数本身也可以赋值给变量，即：变量可以指向函数。且该变量可以来调用这个函数.

### 函数名也是变量
函数名其实就是指向函数的变量. 对于abs()这个函数，完全可以把函数名abs看成变量，它指向一个可以计算绝对值的函数.
如果把abs指向其他对象，会有什么情况发生？
```python
In [25]: abs = 10                                                          
                                                                           
In [26]: abs(-10)                                                          
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-26-c432e3f1fd6c> in <module>()                              
----> 1 abs(-10)                                                           
                                                                           
TypeError: 'int' object is not callable                                    
```
把abs指向10后，就无法通过abs(-10)调用该函数了, 因为abs这个变量已经不指向求绝对值函数而是指向一个整数10！
当然实际代码绝对不能这么写，这里是为了说明函数名也是变量。要恢复abs函数，请重启Python交互环境。
注：由于abs函数实际上是定义在import builtins模块中的，所以要让修改abs变量的指向在其它模块也生效，要用import builtins; builtins.abs = 10。

### 将函数作为参数传入
```python
In [1]: def add(x, y, f):
   ...:     return f(x) + f(y)
   ...:

In [2]: add(2, -3, abs)
Out[2]: 5
```
编写高阶函数，目的就是让函数的参数能够接收别的函数。

### 常用的高阶函数
1. map()高阶函数
map()函数接收两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回.
```python
In [3]: def f(x):
   ...:     return x*x
   ...:

In [4]: r = map(f, [1, 2, 3, 4, 5])

In [5]: list(r)
Out[5]: [1, 4, 9, 16, 25]

In [6]: list(map(str, [1, 2, 3, 4, 5]))
Out[6]: ['1', '2', '3', '4', '5']
```

map()传入的第一个参数是f，即函数对象本身。由于结果r是一个Iterator，Iterator是惰性序列，因此通过list()函数让它把整个序列都计算出来并返回一个list.

2. reduce()高阶函数
reduce把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，一个是函数，一个是Iterable，reduce的作用就是把结果继续和序列的下一个元素做累积计算.即对于序列内所有元素进行累计操作.
```python
In [12]: from functools import reduce

In [13]: def add(x, y):
    ...:     return x+y
    ...:

In [14]: reduce(add, [1, 2, 3, 4])
Out[14]: 10

# 配合map()，我们就可以写出把str转换为int的函数：
>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
...
>>> def char2num(s):
...     return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]
...
>>> reduce(fn, map(char2num, '13579'))
13579
# 更简洁的写法,采用lambda函数
def str2int(s):
    return reduce(lambda x, y: x * 10 + y, map(char2num, s))
```

3. fiter()高阶函数
Python内建的filter()函数用于过滤序列。filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。
```python
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]
```

4. sorted()高阶函数
排序也是在程序中经常用到的算法。排序的核心是比较两个元素的大小。sorted()函数也是一个高阶函数，它接受3个参数, 其中接收一个key函数来实现自定义的排序，key指定的函数将作用于list的每一个元素上，并根据key函数返回的结果进行排序, 例如按绝对值大小排序：
```python
# sorted函数原型
sorted(...)
    sorted(iterable, key=None, reverse=False) --> new sorted list

>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]
```

对字符串排序，是按照ASCII的大小比较的,现在，我们提出排序应该忽略大小写，按照字母序排序。
```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
['about', 'bob', 'Credit', 'Zoo']
```

要进行反向排序，不必改动key函数，可以传入第三个参数reverse=True：
```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']
```

### 参考
* [廖雪峰python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014317849054170d563b13f0fa4ce6ba1cd86e18103f28000)
