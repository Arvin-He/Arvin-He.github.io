---
title: Python之_magic变量和函数
date: 2017-06-09 17:34:35
tags: Python
categories: 编程
---
### 关于下划线(_)的说明
* 单下划线开头：弱内部使用标识，无法被from M import *所引用
* 单下划线结尾：避免和python关键字冲突，可以加个后置下划线,如exec_()
* 双下划线开头：类成员变量中的私有变量，
* 双下划线开头，双下划线结尾：这是magic对象或属性的名字，永远不要将这样的命名方式应用于自己的变量和函数

### magic变量和magic函数
1. `__name__` 变量
`__name__`属性是直接内置在.py文件中的, 这个属性经常用来当做一个使用模式的标识.
* 如果.py文件是在命令行运行的文件, 即被执行的文件，则`__name__`将被设置为`__main__`。
* 如果.py文件是被import，`__name__`将被设置为.py文件的名字

2. `__file__` 变量
`__file__`可以用来获取python脚本的“路径+脚本名称”，这可能是一个相对路径也可能是一个绝对路径，取决按照什么路径来执行的脚本，一般来说`__file__`变量和os.path配合，可以用来获取python脚本的绝对路径：
```python
In [5]: import os
In [7]: print(os.path.realpath(test1.__file__))
C:\Users\aron\Desktop\test1.py
```

3. `__import__` 函数
python导入模块时，一般使用import，而import语句其实也是调用builtin函数,即`__import__()`函数实现的导入，直接使用`__import__`比较少见，除非导入的模块是不确定的，需要在运行时才能确定导入哪些模块，可以使用`__import__`，默认接收需要导入的模块名的字符串：
```python
In [9]: model = __import__('test')
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]

In [11]: model.Fib(5)
Out[11]: <test.Fib at 0x49ea9b0>
```

4. `__str__` 函数
`__str__`是一个比较常用的内置函数，在定义类的时候经常使用，`__str__`函数返回一个字符串，这个字符串就是此对象被print时显示的内容，如果不定义这个函数，将会显示默认的格式：`<__main__.A object at 0x0000000001FB7C50>`.这个函数在django的model类中如果定义的话，print一条数据库中的数据，可以指定显示任何的值.
**注意：**在python3.x中`__str__`被废弃，使用unicode

5. `__init__` 函数
`__init__`比较常见，是对象的初始化函数

6. `__new__` 函数
`__new__()`函数是类创建对象时调用的内置函数，必须返回一个生成的对象，
`__new__()`函数在`__init__()`函数之前执行。一般来说没有必要重载这个函数，除非需要更改new对象的流程,有一种场景“单例模式”要求只能存在一个class A的对象，如果重复创建，那么返回的已经创建过的对象的引用。可以这样使用`__new__`函数.
单例模式
```python
class A(object):
    def __new__(cls):
        if not "_instance" in vars(cls):
            cls._instance=super(A,cls).__new__(cls)
        return cls._instance
a=A()
b=A()
print id(a)==id(b)
out>>True
```

7. `__class__` 变量
`instance.__class__`表示这个对象的类对象，但在python中，类也是一个对象,例：
```python
In [12]: class A(object):
    ...:     pass
    ...:

In [13]: a = A()

In [14]: B = a.__class__

In [15]: B
Out[15]: __main__.A

In [16]: b = B()

In [17]: print(type(b))
<class '__main__.A'>
```

可以看出，a是A类的一个对象，`a.__class__`就是A类，将这个类赋值给B，使用B()又可以创建出一个对象b，这个对象b也是A类的对象，这个`__class__`有什么用呢？请看下面的例子

8. `__add__` 函数
这里其实包含一系列函数，包括`__sub__`，`__mul__`，`__mod__`，`__pow__`，`__xor__`,
这些函数是对加、减、乘、除、乘方、异或、等运算的重载，重写这些函数, 则我们自定义的对象可以具备运算功能.
```python
class A(object):
    def __init__(self,v):
        self.v=v

    def __add__(self,other):
        #创建创建一个新的对象
        x=self.__class__(self.v + 2*other.v)
        return x

a=A(1)
b=A(2)
c=a+b
print c.v
ouot>>5
```

这样我们就自定义了一个加法操作1+2=1+2*2=5

9. `__doc__` 文档字符串变量
python建议在定义一个类、模块、函数的时候定义一段说明文字，以便提取这些变量字符串,方便做成说明文档,例子如下：
```python
#c.py
"""
script c's doc
"""
class A(object):
    """
    class A's doc
    """
    pass
def B():
    """
    function B's doc
    """
    pass

print __doc__
print A.__doc__
print B.__doc__
out>>script c's doc
out>>class A's doc
out>>function B's doc
```

调用别的模块、函数时如果不清楚使用方法，也可以直接查看doc文档字符串.

10. `__iter__` 函数 和 `__next__` 函数
凡是可以被`for...in`的循环调用的对象，我们称之为可以被迭代的对象，list，str，tuple都可以被迭代，它们都实现了内部的迭代器函数，比如说list，tuple，字符串这些数据结构的迭代器如下：
```python
a=[1,2,3,4]
b=('i',1,[1,2,3])
print a.__iter__()
print b.__iter__()
out>><listiterator object at 0x0000000001CC7C50>
out>><tupleiterator object at 0x0000000001CC7B00>
```

如果我们要实现一个我们自己的迭代器对象，那么我们必须实现两个默认的方法:`__iter__`和`__next__`。
`__iter__()`函数将会返回一个迭代器对象，`__next__()`函数每次被调用都返回一个值，如果迭代完毕，则raise一个StopIteration的错误，用来终止迭代。
下面的例子将实现一个可以迭代的对象，输出a~z的26个字母，该对象接收一个int参数用来表示输出字母的数量，如果该参数超过字母表的长度，则循环从‘a-z’再次进行循环输出：
```python
import random
class A(object):
    def __init__(self,n):
        self.stop=n
        self.value=0
        #字母列表
        self.alph=[chr(i) for i in range(97,123)]
    def __iter__(self):
        return self

    def __next__(self):
        #如果超过长度超过26则重置
        if self.value==len(self.alph):
            self.value=0
            self.stop=self.stop-len(self.alph)
        #最终，已完成n个字符的输出，则结束迭代
        if self.value>self.stop:
            raise StopIteration    
        x=self.alph[self.value]
        self.value+=1
        return x

for i in A(1000):
    print i,
out>>a b c d e f g h i j k l m n o p q r s t u v w x y z a b c d e f
 g h i j k l m n o p q r s t u v w x y z a b c d e f g h i j k .....
```

**注意:** `__next__`是python3中的, python2的是next()

11. `__dict__` 变量, `__slot__` 变量 和 `__all__` 变量
这三个变量有一些关系，`__dict__`在类和对象中都存在，它是一个包含变量名和变量的字典，见以下的例子：
```python
#a.py
class A(object):
    c=3
    d=4
    def __init__(self):
        self.a=1
        self.b=2
    def func(self):
        pass

print A().__dict__
print A.__dict__

out>>{'a': 1, 'b': 2}
out>>{'__module__': '__main__', 'd': 4, 'c': 3, 'func': <function func at 0x00000000021F2BA8>, '__dict__': <attribute '__dict__' of 'A' objects>, '__weakref__': <attribute '__weakref__' of 'A' objects>, '__doc__': None, '__init__': <function __init__ at 0x00000000021F2AC8>}
```

一个对象的`__dict__`只包含self定义的变量，
一个类的`__dict__`包含了类里面的函数（func函数）、类变量，以及很多隐性的变量，包括`__dict__`变量本身也是隐性的。

`__slot__`变量的用法理解起来比较要难一点，正常的情况下，我们实例化一个对象，可以给这个对象增加任意的成员变量，即使不在类里面定义的变量都可以. 
```python
#a.py
class A(object):

    def __init__(self):
        self.a=1
        self.b=2

a=A()
#给a增加一个x变量
a.x=1
#也可以给a增加一个匿名函数
a.y=lambda x,y:x*y
print a.x
print a.y(3,5)
out>>1
out>>15
```

但如果我们想限制一下对象绑定的变量，我们可以在类定义的时候增加一个slots变量，这个变量是一个**字符串元组**，例子如下：
```python
class A(object):
    __slots__=('a','b','x')
    def __init__(self):
        self.a=1
        self.b=2

    #__slots__=('a','b',)
    def func(self):
        pass
a=A()
a.x=1
#执行到a.y时会报错：AttributeError: 'A' object has no attribute 'y'
a.y=lambda x,y:x*y
print a.y(3,5)
```

`__all__`变量是一个字符串列表，
它定义了每一个模块会被`from module_name import *`这样的语句可以被import的内容（变量，类，函数）
```python
#a.py 不定义__all__
class A(object):
    def __init__(self):
        self.a=1
        self.b=2

    def func(self):
        pass
def B():
    pass

c=10

#b.py
from a import *
print A
print B
print c
out>><class 'learn_draft.A'>
out>><function B at 0x00000000021D1438>
out>>10
```

如果在a.py中定义`__all__=['A','c']`,则B函数对于b.py来说是不可见的

12. `__hash__` 函数
哈希函数，在python中的对象有一个hashable（可哈希）的概念.
对于数字、字符串、元组来说，是不可变的，也就是可哈希的，因此这些对象也可以作为字典的key值。
对于列表、字典等，是可变对象，因此是不可哈希的，也就不能作为字典的key值。
是否可哈希，可以调用内置函数`hash()`进行计算，`hash()`函数返回计算的到的hash值。
完全相同的变量，调用哈希算法的到的hash值一定是相同的

当然一般来说，我们不会去重新定义一个对象的`__hash__`函数，除非我们想实现一个自定义的需求，
在stackoverflow有人提出这样一个需求，需要判断有相同词频的字符串是相等的，也就是说“abb”和“bab”这样的字符串是相等的，这个时候我们可以继承字符串类，然后重写哈希函数，如下：
```python
import collections

class FrequencyString(str):
    @property
    def normalized(self):
        try:
            return self._normalized
        except AttributeError:
            self._normalized = normalized = ''.join(sorted(collections.Counter(self).elements()))
            return normalized

    def __eq__(self, other):
        return self.normalized == other.normalized

    def __hash__(self):
        return hash(self.normalized)
```

13. `__getattr__` 函数和`__setattr__` 函数，`__delattr__` 函数 
先介绍两个内置函数，`getattr()`和`setattr()`,使用这两个函数可以获取对象的属性，或者给对象的属性赋值：
```python
#a.py
class A(object):
    def __init__(self):
        self.a=1
        self.b=2
a=A()
setattr(a,'a',3)
print a.a
print getattr(a,'b')
out>>3
out>>2
```

其实使用这两个函数和直接访问`a.a`, `a.b`没有任何区别，但好处是setattr和getattr接受两个字符串去确定访问对象a的哪一个属性，和`__import__`一样，可以在运行时在决定去访问对象变量的名字，在实际工作中经常会使用这两个函数。

`__getattr__()`这个函数是在访问对象不存在的成员变量是才会访问的，见下面的例子：
```python
class A(object):
    def __init__(self):
        self.a=1
        self.b=2

    def func(self):
        pass

    def __getattr__(self,name):
        print 'getattr'
        return self.a

a=A()
print a.d
out>>getattr
out>>1
```

在调用a.d时，d不是a的成员变量，则python会去查找对象是否存在`__getattr__()`函数，如果存在，则返回`__getattr__()`函数的返回值，我们这里返回的是self.a的值1。

由于`__getattr__()`的特性，我们可以将`__getattr__()`设计成一个公共的接口函数，在autotest的`proxy.py`中就看到了这样的用法：
```python
class ServiceProxy(object):

def __init__(self, serviceURL, serviceName=None, headers=None):
    self.__serviceURL = serviceURL
    self.__serviceName = serviceName
    self.__headers = headers or {}

def __getattr__(self, name):
    if self.__serviceName is not None:
        name = "%s.%s" % (self.__serviceName, name)
    return ServiceProxy(self.__serviceURL, name, self.__headers)

#调用的时候，op是执行的特定操作的字符串，op传入__getattr__将会把ServiceProxy对象重新的内部变量重新赋值，然后返回一个更新之后的对象
function = getattr(self.proxy, op)
```

`__setattr__`和`__getattr__`不一样，对象的所有属性赋值，都会经过`__setattr__()`函数，看下面的例子：
```python
class A(object):
    def __init__(self):
        self.a=1
        self.b=2

    def func(self):
        pass

    def __getattr__(self,name):
        print 'getattr'
        return self.a

    def __setattr__(self, name, value):
        print 'setattr %s' % name
        if name == 'f':
            return object.__setattr__(self,name,value+1000)
        else:
            return object.__setattr__(self,  name, value)

a=A()
a.f=1000
print a.f
out>>setattr a
out>>setattr b
out>>setattr f
out>>2000
```

从输出可以看到init函数的self.a和self.b的赋值也经过了`__setattr__`，而且在赋值的时候我们自定义了一个if逻辑，如果name是‘f’，那么value会增加1000，最终的a.f是2000

`__delattr__`是删除一个对象属性用的。

14. `__call__` 函数
如果一个对象实现了`__call__()`函数，那么这个对象可以认为是一个函数对象，使用加括号的方法就可以调用，见下面例子：
```python
class A(object):
    def __init__(self):
        self.li=['a','b','c','d']

    def func(self):
        pass

    def __call__(self,n):
        #返回li列表的第n个元素
        return self.li[n]

a=A()
#a可以当做函数一样调用
print a(0),a(1),a(2)
out>>a b c
```

在实际工作中`__call__`函数非常有用，可以把一个对象变成callable的对象.

### 参考
* [http://www.jianshu.com/p/1112320c5784](http://www.jianshu.com/p/1112320c5784)
