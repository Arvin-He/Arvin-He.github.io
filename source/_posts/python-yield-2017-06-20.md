---
title: Python之yield与yield from理解
date: 2017-06-20 15:24:39
tags: python
categories: python
---
### yield
为了理解什么是 yield, 你必须理解什么是生成器(generator)。
在函数中使用yield关键字，函数就变成了一个generator。
函数里有了yield后，执行到yield就会停住，当需要再往下算时才会再往下算。所以生成器函数即使是有无限循环也没关系，它需要算到多少就会算多少，不需要就不往下算。

### yield from
`yield from <expr>` 其中`<expr>`是一个可迭代的表达式，自动调用 iter()，获取迭代器。所以要求`<expr>`是可迭代对象。此外, yield from是可以实现嵌套生成器的使用.

yield from 包含几个概念：
* 委派生成器: 包含yield from表达式的生成器函数
* 子生成器: 从yield from部分获取的生成器。
* 调用方: 调用委派生成器的(调用方)代码

yield from 可用于简化for循环中的yield表达式。

yield from 是 Python3.3 后新加的语言结构。和其他语言的await关键字类似，
它表示：在生成器 gen 中使用 `yield from subgen()`时，subgen 会获得控制权，
把产出的值传个gen的调用方，即调用方可以直接控制subgen。于此同时，gen会阻塞，等待subgen执行完毕, 然后在接着运行gen.
```python
In [1]: def gen():
   ...:     for c in 'AB':
   ...:         yield c
   ...:     for i in range(1, 3):
   ...:         yield i
   ...:

In [2]: list(gen())
Out[2]: ['A', 'B', 1, 2]
```

改写为:
```python
In [3]: def gen():
   ...:     yield from 'AB'
   ...:     yield from range(1, 3)
   ...:

In [4]: list(gen())
Out[4]: ['A', 'B', 1, 2]
```

下面看一个cookbook的例子:
```python
# Example of flattening a nested sequence using subgenerators
from collections import Iterable

def flatten(items, ignore_types=(str, bytes)):
    for x in items:
        if isinstance(x, Iterable) and not isinstance(x, ignore_types):
            yield from flatten(x) # 这里递归调用，如果x是可迭代对象，继续分解
        else:
            yield x

items = [1, 2, [3, 4, [5, 6], 7], 8]

# Produces 1 2 3 4 5 6 7 8

for x in flatten(items):
    print(x)

items = ['Dave', 'Paula', ['Thomas', 'Lewis']]
for x in flatten(items):
    print(x)    
```

yield from 的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来，使两者可以直接发送和产出值，还可以直接传入异常，而不用在中间的协程添加异常处理的代码。

委派生成器在 yield from 表达式处暂停时，调用方可以直接把数据发给子生成器，子生成器再把产出的值发送给调用方。子生成器返回之后，解释器会抛出StopIteration异常，并把返回值附加到异常对象上，指示委派生成器恢复。

此外，当迭代器是另一个生成器时，允许子生成器执行return语句返回一个值，该值将成为yield from表达式的值。

PEP380 分6点说明了yield from 的行为:
* 子生成器产出的值都直接传给委派生成器的调用方
* 使用send() 方法发给委派生成器的值都直接传给迭代器。如果发送的值是None，那么会调用子迭代器的 `__next__()`方法。如果发送的值不是None，那么会调用迭代器的send()方法。如果调用的方法抛出StopIteration异常，那么委派生成器恢复运行。任何其他异常都传给委派生成器。
* 引发委托生成器的GeneratorExit以外的异常传递给迭代器的throw（）方法。如果调用throw()方法时抛出 StopIteration 异常，委派生成器恢复运行。StopIteration之外的异常会向上传给委派生成器。
* 如果把 GeneratorExit 异常传入委派生成器，或者在委派生成器上调用close() 方法，那么在迭代器上调用close() 方法，如果迭代器有的话。如果调用close() 方法导致异常抛出，那么异常会向上传给委派生成器；否则，委派生成器抛出 GeneratorExit 异常。
* yield from表达式的值是迭代器终止时迭代器抛出StopIteration异常的第一个参数。
* 生成器退出时，生成器（或子生成器）中的return expr 表达式会触发 StopIteration(expr) 异常抛出。


### 参考
* [pep-0380](https://www.python.org/dev/peps/pep-0380/#proposal)