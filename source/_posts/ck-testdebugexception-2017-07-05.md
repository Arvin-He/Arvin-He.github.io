---
title: cookbook之测试,调试和异常
date: 2017-07-05 16:47:21
tags: Python
categories: 编程
---
### 在单元测试中给对象打补丁
写的单元测试中需要给指定的对象打补丁，用来断言它们在测试中的期望行为（比如，断言被调用时的参数个数，访问指定的属性等）
unittest.mock.patch() 函数可被用来解决这个问题。 patch() 还可被用作一个装饰器、上下文管理器或单独使用，尽管并不常见。

### 将测试输出用日志记录到文件中
```python
import unittest

class MyTest(unittest.TestCase):
    pass

if __name__ == '__main__':
    unittest.main()
```

这样的话测试文件就是可执行的，并且会将运行测试的结果打印到标准输出上。如果你想重定向输出，就需要像下面这样修改 main() 函数：
```python
import sys

def main(out=sys.stderr, verbosity=2):
    loader = unittest.TestLoader()
    suite = loader.loadTestsFromModule(sys.modules[__name__])
    unittest.TextTestRunner(out,verbosity=verbosity).run(suite)

if __name__ == '__main__':
    with open('testing.out', 'w') as f:
        main(f)
```

### 忽略或期望测试失败
在单元测试中忽略或标记某些测试会按照预期运行失败。unittest 模块有装饰器可用来控制对指定测试方法的处理
```python
import unittest
import os
import platform

class Tests(unittest.TestCase):
    def test_0(self):
        self.assertTrue(True)

    @unittest.skip('skipped test')
    def test_1(self):
        self.fail('should have failed!')

    @unittest.skipIf(os.name=='posix', 'Not supported on Unix')
    def test_2(self):
        import winreg

    @unittest.skipUnless(platform.system() == 'Darwin', 'Mac specific test')
    def test_3(self):
        self.assertTrue(True)

    @unittest.expectedFailure
    def test_4(self):
        self.assertEqual(2+2, 5)

if __name__ == '__main__':
    unittest.main()
```

kip() 装饰器能被用来忽略某个你不想运行的测试。 skipIf() 和 skipUnless()对于你只想在某个特定平台或 Python 版本或其他依赖成立时才运行测试的
时候非常有用。使用 @expected 的失败装饰器来标记那些确定会失败的测试，并且对这些测试你不想让测试框架打印更多信息。

### 处理多个异常
有一个代码片段可能会抛出多个不同的异常,可以用单个代码块处理不同的异常，可以将它们放入一个元组中
```python
try:
    client_obj.get_url(url)
except (URLError, ValueError, SocketTimeout):
    client_obj.remove_url(url)
```

如果你想对其中某个异常进行不同的处理，可以将其放入另外一个 except 语句中：
```python
try:
    client_obj.get_url(url)
except (URLError, ValueError):
    client_obj.remove_url(url)
except SocketTimeout:
    client_obj.handle_url_timeout(url)
```

很多的异常会有层级关系，对于这种情况，你可能使用它们的一个基类来捕获所有的异常。
```python
try:
    f = open(filename)
except (FileNotFoundError, PermissionError):
    pass

# 用基类来捕获所有的异常
try:
    f = open(filename)
except OSError:
    pass
```    

使用 as 关键字来获得被抛出异常的引用
```python
try:
    f = open(filename)
except OSError as e:
    if e.errno == errno.ENOENT:
        logger.error('File not found')
    elif e.errno == errno.EACCES:
        logger.error('Permission denied')
    else:
        logger.error('Unexpected error: %d', e.errno)
```

### 捕获所有异常
捕获所有的异常，可以直接捕获 Exception 即可, 这个将会捕获除了 SystemExit 、 KeyboardInterrupt 和 GeneratorExit 之外的
所有异常。如果你还想捕获这三个异常，将 Exception 改成 BaseException 即可.
```python
try:
    ...
except Exception as e:
    ...
    log('Reason:', e) # Important!
```

### 捕获异常后抛出另外的异常
链接异常，使用 raise from 语句来代替简单的 raise 语句, 同时保留两个异常的信息
```python
>>> def example():
... try:
...     int('N/A')
... except ValueError as e:
...     raise RuntimeError('A parsing error occurred') from e
>>>
example()
Traceback (most recent call last):
File "<stdin>", line 3, in example
ValueError: invalid literal for int() with base 10: 'N/A'
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
File "<stdin>", line 5, in example
RuntimeError: A parsing error occurred
```

### 输出警告信息
在你维护软件，提示用户某些信息，但是又不需要将其上升为异常级别，那么输出警告信息就会很有用了.
希望程序能生成警告信息（比如废弃特性或使用问题）,可使用 warning.warn() 函数.
```python
import warnings

def func(x, y, logfile=None, debug=False):
    if logfile is not None:
        warnings.warn('logfile argument deprecated', DeprecationWarning)
```

warn() 的参数是一个警告消息和一个警告类，警告类有如下几种：
UserWarning, DeprecationWarning, SyntaxWarning, RuntimeWarning, ResourceWarning, 或 Future-Warning.
对警告的处理取决于你如何运行解释器以及一些其他配置。例如，如果你使用 -W all 选项去运行 Python，你会得到如下的输出：
```
bash % python3 -W all example.py
example.py:5: DeprecationWarning: logfile argument is deprecated
warnings.warn('logfile argument is deprecated', DeprecationWarning)
```

通常来讲，警告会输出到标准错误上。如果你想讲警告转换为异常，可以使用 -W error 选项：
```
bash % python3 -W error example.py
Traceback (most recent call last):
File "example.py", line 10, in <module>
func(2, 3, logfile='log.txt')
File "example.py", line 5, in func
warnings.warn('logfile argument is deprecated', DeprecationWarning)
DeprecationWarning: logfile argument is deprecated
```

默认情况下，并不是所有警告消息都会出现。-W 选项能控制警告消息的输出。 -W all 会输出所有警告消息，-W ignore 忽略掉所有警告，-W error 将警告转换成异常。
另外一种选择，你还可以使用 warnings.simplefilter() 函数控制输出。 always 参数会让所有警告消息出现，`ignore 忽略调所有的警告，error 将警告转换成异常。
warnings 模块对过滤和警告消息处理提供了大量的更高级的配置选项。

### 调试基本的程序崩溃错误
程序奔溃后该怎样去调试它？
运行 python3 -i someprogram.py 可执行简单的调试。 -i 选项可让程序结束后打开一个交互式 shell。然后你就能查看环境.
可以在程序奔溃后打开 Python 的调试器
```python
>>> import pdb
>>> pdb.pm()
> sample.py(4)func()
-> return n + 10
(Pdb) w
sample.py(6)<module>()
-> func('Hello')
> sample.py(4)func()
-> return n + 10
(Pdb) print n
'Hello'
(Pdb) q
>>>
```

代码所在的环境很难获取交互 shell（比如在某个服务器上面），通常可以捕获异常后自己打印跟踪信息
```python
import traceback
import sys
try:
    func(arg)
except:
    print('**** AN ERROR OCCURRED ****')
    traceback.print_exc(file=sys.stderr)
```

要是你的程序没有奔溃，而只是产生了一些你看不懂的结果，你在感兴趣的地方插入一下 print() 语句也是个不错的选择。
不过，要是你打算这样做，有一些小技巧可以帮助你。首先，traceback.print stack() 函数会你程序运行到那个点的时候创建
一个跟踪栈。
另外，你还可以像下面这样使用 pdb.set trace() 在任何地方手动的启动调试器
```python
import pdb
def func(arg):
    ...
    pdb.set_trace()
    ....
```

### 给程序做性能测试
测试程序运行所花费的时间并做性能测试。
只是简单的想测试下你的程序整体花费的时间，通常使用 Unix 时间函数就行了.
需要一个程序各个细节的详细报告，使用 cProfile 模块.
通常情况是介于这两个极端之间。比如你已经知道代码运行时在少数几个函数中花费了绝大部分时间。对于这些函数的性能测试，可以使用一个简单的装饰器：
```python
# timethis.py
import time
from functools import wraps

def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        r = func(*args, **kwargs)
        end = time.perf_counter()
        print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
        return r
    return wrapper

>>> @timethis
... def countdown(n):
... while n > 0:
... n -= 1
...
>>> countdown(10000000)
__main__.countdown : 0.803001880645752
```

要测试某个代码块运行时间，你可以定义一个上下文管理器
```python
from contextlib import contextmanager

@contextmanager
def timeblock(label):
    start = time.perf_counter()
    try:
        yield
    finally:
        end = time.perf_counter()
        print('{} : {}'.format(label, end - start))

>>> with timeblock('counting'):
... n = 10000000
... while n > 0:
... n -= 1
...
counting : 1.5551159381866455
```

对于测试很小的代码片段运行性能，使用 timeit 模块会很方便
```python
>>> from timeit import timeit
>>> timeit('math.sqrt(2)', 'import math')
0.1432319980012835
>>> timeit('sqrt(2)', 'from math import sqrt')
0.10836604500218527

>>> timeit('math.sqrt(2)', 'import math', number=10000000)
1.434852126003534
>>> timeit('sqrt(2)', 'from math import sqrt', number=10000000)
1.0270336690009572
```

timeit 会执行参数中语句 100 万次并计算运行时间。第二个参数是运行测
试之前配置环境。如果你想改变循环执行次数，可以设置 number 参数.

当执行性能测试的时候，需要注意的是你获取的结果都是近似值。`time.perf_counter()` 函数会在给定平台上获取最高精度的计时值。不过，它仍然
还是基于时钟时间，很多因素会影响到它的精确度，比如机器负载。如果你对于cpu执行时间更感兴趣，使用 `time.process_time()` 来代替它。
```python
from functools import wraps

def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.process_time()
        r = func(*args, **kwargs)
        end = time.process_time()
        print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
        return r
    return wrapper
```

### 加速程序运行
你的程序运行太慢，你想在不使用复杂技术比如 C 扩展或 JIT 编译器的情况下加快程序运行速度.
关于程序优化的第一个准则是“不要优化”，第二个准则是“不要优化那些无关紧要的部分”。
通常会发现你得程序在少数几个热点地方花费了大量时间
1. 使用函数
定义在全局范围的代码运行起来要比定义在函数中运行慢的多。这种速度差异是由于局部变量和全局变量的实现方式（使用局部变量要更快些）。因此，如果你想让程序运行更快些，只需要将脚本语句放入函数中即可,速度的差异取决于实际运行的程序，不过根据经验，使用函数带来 15-30% 的性能提升是很常见的。
2. 尽可能去掉属性访问
每一次使用点 (.) 操作符来访问属性的时候会带来额外的开销。它会触发特定的方法，比如 `__getattribute__ ()` 和 `__getattr__ ()` ，这些方法会进行字典操作操作。
可以使用 from module import name 这样的导入形式，以及使用绑定的方法.
```python
# 方式1
import math
math.sqrt(n)
# 方式2
from math import sqrt
```

方式2消除了属性访问,用sqrt() 代替了 math.sqrt() 

3. 理解局部变量
局部变量会比全局变量运行速度快,在内部循环中，可以将某个需要频繁访问的属性放入到一个局部变量中.对于类中的属性访问也同样适用于这个原理。通常来讲，查找某个值比如
self.name 会比访问一个局部变量要慢一些。
4. 避免不必要的抽象
任何时候当你使用额外的处理层（比如装饰器、属性访问、描述器）去包装你的代码时，都会让程序运行变慢.
5. 使用内置的容器
内置的数据类型比如字符串、元组、列表、集合和字典都是使用 C 来实现的，运行起来非常快。如果你想自己实现新的数据结构（比如链接列表、平衡树等），那么要想在性能上达到内置的速度几乎不可能，因此，还是乖乖的使用内置的吧.
6. 避免创建不必要的数据结构或复制
理解或信任 Python 的内存模型，不要滥用 copy.deepcopy() 之类的函数

作为一般准则，不要对程序的每一个部分都去优化, 因为这些修改回导致代码难以阅读和理解。你应该专注于优化产生性能瓶颈的地方，比如内部循环。
引用John Ousterhout 说过的话：“最好的性能优化时从不工作到工作状态的迁移”。直到你真的需要优化的时候再去考虑它。确保你程序正确的运行通常比让它运行更快要更重要一些（至少开始是这样的）.