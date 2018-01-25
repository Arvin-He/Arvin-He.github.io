---
title: Python之闭包
date: 2017-06-01 08:31:07
tags: python
categories: python
---
### 前言
谈到闭包,先要了解两个概念:作用域和嵌套函数
Python 里面有四种作用域：function, module, global和 class 作用域。由于 Python 不区分变量的声明，所以在第一次初始化变量时（必须为赋值操作）将变量加入当前环境中。如果在没对变量进行初始化的情况下使用该变量就会报运行时异常，但如果仅仅是访问（并不赋值）的情况下，查找变量的顺序会按照 LEGB 规则 (Local, Enclosing, Global, Built-in)。

### 作用域
作用域是程序运行时变量可被访问的范围，定义在函数内的变量是局部变量，局部变量的作用范围只能是函数内部范围内，它不能在函数外引用。
定义在模块最外层的变量是全局变量，它是全局范围内可见的，当然在函数里面也可以读取到全局变量的

```python
s = "hello"
def foo():
    s += "world"
    return s

foo()  # UnboundLocalError: local variable 's' referenced before assignment
```
由于在函数 foo 中在没有对 s 初始化的情况下使用了该值，所以这里会报异常，解决的办法就是使用 global 关键字：
```python
s = "hello"
def foo():
    global s
    s += " world"
    return s
foo()  # return "hello world"
```
但由于 global 关键字只能限定在global作用域内查找变量，在有嵌套定义的时候就有问题了，比如：
```python
def foo():
    s = "hello"
    def bar():
        global s     # NameError: global name 's' is not defined
        s += " world"
        return s
    return bar
foo()()
```
Python 3 中引入了 nonlocal 关键字来解决这个问题，：
```python
def foo():
    s = "hello"
    def bar():
        nonlocal s
        s += " world"
        return s
    return bar
foo()()   # return "hello world"
```

### 嵌套函数
函数不仅可以定义在模块的最外层，还可以定义在另外一个函数的内部，像这种定义在函数里面的函数称之为嵌套函数（nested function）例如：
```python
def print_msg():
    # print_msg 是外围函数
    msg = "zen of python"

    def printer():
        # printer是嵌套函数
        print(msg)
    printer()
# 输出 zen of python
print_msg()
```
对于嵌套函数，它可以访问到其外层作用域中声明的非局部（non-local）变量，比如代码示例中的变量 msg 可以被嵌套函数 printer 正常访问。
那么有没有一种可能即使脱离了函数本身的作用范围，局部变量还可以被访问得到呢？答案是闭包

### 闭包
简单的说:闭包是一个函数内部嵌套着另一个函数，而被嵌套的那个函数有权利访问嵌套它的那个函数的作用域中变量。

当符合下面几个条件时就形成了闭包：
* 有一个Nested function
* 这个Nested function访问了父函数作用域中的变量
* 父函数返回了这个Nested function

```python
def print_msg():
    # print_msg 是外围函数
    msg = "zen of python"
    def printer():
        # printer 是嵌套函数
        print(msg)
    return printer

another = print_msg()
# 输出 zen of python
another()
```
这段代码和上面的嵌套函数的效果一样，都输出 "zen of python"。不同的地方在于内部函数 printer 直接作为返回值返回了。
一般情况下，函数中的局部变量仅在函数的执行期间可用，一旦 `print_msg()` 执行过后，我们会认为 msg变量将不再可用。
然而，在这里我们发现 print_msg 执行完之后，在调用 another 的时候 msg 变量的值正常输出了，这就是闭包的作用，闭包使得局部变量在函数外被访问成为可能。这里的 another 就是一个闭包，闭包本质上是一个函数，它有两部分组成，printer 函数和变量 msg。闭包使得这些变量的值始终保存在内存中。
闭包，顾名思义，就是一个封闭的包裹，里面包裹着自由变量，就像在类里面定义的属性值一样，自由变量的可见范围随同包裹，哪里可以访问到这个包裹，哪里就可以访问到这个自由变量.

### 为什么要使用闭包
运用闭包可以避免对全局变量的使用。对于一个只有需要实现少数方法的类我们也可以用闭包来替代，这样做可以减少资源的使用。
此外，闭包允许将函数与其所操作的某些数据（环境）关连起来。这一点与面向对象编程是非常类似的，在面对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。一般来说，当对象中只有一个方法时，这时使用闭包是更好的选择。来看一个例子：
```python
In [5]: def adder(x):                        
   ...:     def wrapper(y):                  
   ...:         print("x = {}".format(x))    
   ...:         print("y = {}".format(y))    
   ...:         return x+y                   
   ...:     return wrapper                   
   ...:                                      
   ...:                                      
                                             
In [6]: adder5 = adder(5)                    
                                             
In [7]: adder5(10)                           
x = 5                                        
y = 10                                       
Out[7]: 15                                   
                                             
In [8]: adder5(6)                            
x = 5                                        
y = 6                                        
Out[8]: 11                                   
```
这比用类来实现更优雅，此外装饰器也是基于闭包的一中应用场景。
所有函数都有一个 `__closure__`属性，如果这个函数是一个闭包的话，那么它返回的是一个由 cell 对象 组成的元组对象。cell 对象的`cell_contents` 属性就是闭包中的自由变量。这解释了为什么局部变量脱离函数之后，还可以在函数之外被访问的原因的，因为它存储在了闭包的 `cell_contents`中了。

```python
In [9]: adder.__closure__

In [10]: adder5.__closure__
Out[10]: (<cell at 0x03F8FB30: int object at 0x1DFAD210>,)

In [11]: adder5.__closure__[0].cell_contents
Out[11]: 5
```

### 闭包与装饰器
闭包通常用来实现一个通用的功能，Python中的装饰器就是对闭包的一种应用，只不过装饰器中父函数的参数是一个函数
```python
def make_wrap(func):
    def wrapper(*args):
        print("before function")
        func(*args)
        print("after function")
    return wrapper
    
@make_wrap
def print_msg(msg):
    print(msg)
    
>>> print_msg("Hello")
before function
Hello
after function
```
装饰器也可以进行叠加
```python
def make_another(func):
    def wrapper(*args):
        print("another begin")
        func(*args)
        print("another end")
    return wrapper

@make_another
@make_wrap
def print_msg(msg):
    print(msg)

>>> print_msg("Hello")
another begin
before function
Hello
after function
another end
```
### 参考
* [http://liujiacai.net/blog/2016/05/28/scope-closure/](http://liujiacai.net/blog/2016/05/28/scope-closure/)
* [https://juejin.im/post/59191552128fe1005ccd015f](https://juejin.im/post/59191552128fe1005ccd015f)
* [https://juejin.im/post/59022e730ce463006156fdf3](https://juejin.im/post/59022e730ce463006156fdf3)
* [https://kasheemlew.github.io/2017/04/05/python-closure/](https://kasheemlew.github.io/2017/04/05/python-closure/)