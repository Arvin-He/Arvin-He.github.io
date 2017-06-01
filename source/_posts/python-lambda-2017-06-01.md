---
title: Python之Lambda使用
date: 2017-06-01 10:21:48
tags: Python
categories: 编程
---
### Lambda 函数
Python 中定义函数有两种方法，一种是用常规方式 def 定义，函数要指定名字，第二种是用 lambda 定义，不需要指定名字，称为 Lambda 函数。
Lambda 函数又称匿名函数，匿名函数就是没有名字的函数。有些函数如果只是临时一用，而且它的业务逻辑也很简单时，就没必要非给它取个名字不可。

### lamdba 函数使用场景
1. 函数式编程
Python提供了函数式编程的特性，如 map、reduce、filter、sorted 这些函数都支持函数作为参数.
lambda 函数就可以应用在函数式编程中。下面看一个例子: 将一个整数列表，要求按照列表中元素的绝对值大小升序排列
```python
# 方法1
In [12]: my_list = [2, 6, 5, -3, -1, 8]           
                                                  
In [13]: sorted(my_list, key=lambda x : abs(x))   
Out[13]: [-1, 2, -3, 5, 6, 8]                     

# 方法2                                              
In [14]: def foo(x):                              
    ...:     return abs(x)                        
    ...:                                          
                                                  
In [15]: sorted(my_list, key=foo)                 
Out[15]: [-1, 2, -3, 5, 6, 8]                                        
```
方法1和方法2都能达到同样的效果,只不过方法2看起来不够 Pythonic 而已。方法1更加简洁

2. 闭包
看一个用 lambda 函数作为闭包的例子
```python
>>> def my_add(n):
...     return lambda x:x+n
...
>>> add_3 = my_add(3)
>>> add_3(7)
10

>>> def my_add(n):
...     def wrapper(x):
...         return x+n
...     return wrapper
...
>>> add_5 = my_add(5)
>>> add_5(2)
7
```
这里的 lambda 函数就是一个闭包，在全局作用域范围中，`add_3(7)` 可以正常执行且返回值为10，之所以返回10是因为在`my_add` 局部作用域中，变量 n 的值在闭包的作用使得它在全局作用域也可以被访问到。换成常规函数也可以实现闭包，只不过是这种方式稍显啰嗦。

### 使用lambda局限与注意点
1. python中的lambda中不可以赋值
2. python中的lambda只能写一句表达式,不可以写多个语句
3. 如果用 lambda 函数不能使你的代码变得更清晰时，这时就要考虑使用常规的方式来定义函数, zen of python 中有这样一句话是 Explicit is better than implicit(明了胜于晦涩)
