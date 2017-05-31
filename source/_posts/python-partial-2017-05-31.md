---
title: Python之functools
date: 2017-05-31 16:57:57
tags: Python
categories: 编程
---
### functools.partial
functools.partial 通过包装手法，允许我们 "重新定义" 函数签名, 用一些默认参数包装一个可调用对象, 返回结果是可调用对象，并且可以像原始对象一样对待冻结部分函数位置函数或关键字参数，简化函数, 更少更灵活的函数参数调用. 总之,通过设定参数的默认值，可以降低函数调用的难度.
简单总结functools.partial的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。
```python
import functools

def add(a, b):
    return a + b

add(4, 2)
6

plus3 = functools.partial(add, 3)
plus3(4)
7
```

### functool.update_wrapper
默认partial对象没有`__name__`和`__doc__`, 这种情况下，对于装饰器函数非常难以debug.
使用`update_wrapper()`,从原始对象拷贝或加入现有partial对象,它可以把被封装函数的`__name__`、 `__module__` 、`__doc__`和` __dict__`都复制到封装函数去(模块级别常量`WRAPPER_ASSIGNMENTS`, `WRAPPER_UPDATES`)
这个函数主要用在装饰器函数中，装饰器返回函数反射得到的是包装函数的函数定义而不是原始函数定义
```python
from functools import update_wrapper
def wrap2(func):
    def call_it(*args, **kwargs):
        """wrap func: call_it2"""
        print 'before call'
        return func(*args, **kwargs)
    return update_wrapper(call_it, func)

@wrap2
def hello2():
    """test hello"""
    print 'hello world2'

if __name__ == '__main__':

    hello2()
    print hello2.__name__
    print hello2.__doc__

before call
hello world2
hello2
test hello
```

### functool.wraps
调用函数装饰器partial(update_wrapper, wrapped=wrapped, assigned=assigned, updated=updated)的简写
```python
from functools import wraps
def wrap3(func):
    @wraps(func)
    def call_it(*args, **kwargs):
        """wrap func: call_it2"""
        print 'before call'
        return func(*args, **kwargs)
    return call_it

@wrap3
def hello3():
    """test hello 3"""
    print 'hello world3'


before call
hello world3
hello3
test hello 3
```

### functools.reduce
等同于内置函数reduce(),用这个的原因是使代码更兼容(python3)

### functools.cmp\_to\_key
`functools.cmp_to_key(func)`
将老式的比较函数（comparison function）转换为关键字函数（key function），与接受key function的工具一同使用（例如sorted，min，max，heapq.nlargest，itertools.groupby），该函数主要用于将程序转换成Python 3格式的，因为Python 3中不支持比较函数。
```python
from functools import cmp_to_key

def compare(ele1,ele2):
    return ele2 - ele1

a = [2,3,1]
print sorted(a, key = cmp_to_key(compare))

[3, 2, 1]
```

### functools.total\_ordering
`functools.total_ordering(cls)`这是一个类装饰器，给定一个类，这个类定义了一个或者多个比较排序方法，这个类装饰器将会补充其余的比较方法，减少了自己定义所有比较方法时的工作量；
这个装饰器是在python2.7的时候加上的，它是针对某个类如果定义了`__lt__`、 le 、 gt 、`__ge__`这些方法中的至少一个，同时，被修饰的类还应该提供 `__eq__()`方法。使用该装饰器，则会自动的把其他几个比较函数也实现在该类中

```python
@total_ordering
class Student:
    def __eq__(self, other):
        return ((self.lastname.lower(), self.firstname.lower()) ==
                (other.lastname.lower(), other.firstname.lower()))
    def __lt__(self, other):
        return ((self.lastname.lower(), self.firstname.lower()) <
                (other.lastname.lower(), other.firstname.lower()))
print dir(Student)

['__doc__', '__eq__', '__ge__', '__gt__', '__le__', '__lt__', '__module__']
```
### 参考
* [http://www.tuicool.com/articles/qEjEBzy](http://www.tuicool.com/articles/qEjEBzy)