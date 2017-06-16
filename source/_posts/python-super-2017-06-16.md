---
title: Python之super理解
date: 2017-06-16 15:29:08
tags: [Python]
categories: 编程
---
### super使用场景
在类的继承中，如果重写某个方法，该方法会覆盖父类的同名方法，但有时，我们希望能调用子类的方法的同时也能调用父类的同名方法，这时，可通过使用 super 来实现，比如：
```python
class Animal(object):
    def __init__(self, name):
        self.name = name

    def greet(self):
        print('Hello, I am %s.' % self.name)

class Dog(Animal):
    def greet(self):
        super(Dog, self).greet()   # Python3 可使用 super().greet()
        print('WangWang...')

```     

在上面，Animal 是父类，Dog 是子类，我们在 Dog 类重定义了 greet 方法，为了能同时实现父类的功能，我们又调用了父类的方法，看下面的使用：
```python
>>> dog = Dog('dog')
>>> dog.greet()
Hello, I am dog.
WangWang..
```

### super最常用的用法
super 的一个最常见用法可以说是在子类中调用父类的初始化方法了，比如：   
```python
class Base(object):
    def __init__(self, a, b):
        self.a = a
        self.b = b

class A(Base):
    def __init__(self, a, b, c):
        # Python3 可使用 super().__init__(a, b)
        super(A, self).__init__(a, b) 
        self.c = c
```       

### 深入理解super
看了上面的例子，你可能会觉得 super 的使用就是获取了父类，并调用父类的方法。其实，在上面的情况下，super 获得的类刚好是父类，但在其他情况就不一定了，super 其实和父类没有实质性的关联。
下面看一个稍微复杂的例子，涉及到多重继承，代码如下：
```python
class Base(object):
    def __init__(self):
        print("enter Base")
        print("leave Base")

class A(Base):
    def __init__(self):
        print("enter A")
        super(A, self).__init__()
        print("leave A")

class B(Base):
    def __init__(self):
        print("enter B")
        super(B, self).__init__()
        print("leave B")

class C(A, B):
    def __init__(self):
        print("enter C")
        super(C, self).__init__()
        print("leave C")
```

实例化子类C看看结果:
```python
>>> c = C()
enter C
enter A
enter B
enter Base
leave Base
leave B
leave A
leave C
```

如果 super 代表『调用父类的方法』，那么为什么 enter A 的下一句不是 enter Base 而是 enter B。
原因是: super 和父类没有实质性的关联，下面请看 super 是怎么运作的。

### MRO列表
事实上，对于你定义的每一个类，Python 会计算出一个方法解析顺序列表(Method Resolution Order, MRO)，它表明了类继承的顺序，我们使用下面的方法获得某个类的 MRO 列表：
```python
>>> C.mro()   # or C.__mro__ or C().__class__.mro()
[__main__.C, __main__.A, __main__.B, __main__.Base, object]
```

那么这个 MRO 列表的顺序是怎么定的呢?
它是通过一个 C3 线性化算法来实现的，这里不深究这个算法，感兴趣的可以自己去了解，总之，一个类的 MRO 列表就是合并**所有父类**的 MRO 列表，并遵循以下三条原则：
1. 子类永远在父类前面
2. 如果有多个父类，会根据它们在列表中的顺序被检查
3. 如果对下一个类存在两个合法的选择，选择第一个父类

### super原理及实现
super 的定义如下：
```python
def super(cls, inst):
    mro = inst.__class__.mro()
    return mro[mro.index(cls) + 1]
```

其中, 第一个参数 cls 代表类, 第二个参数 inst 代表实例, 上面的代码做了两件事:
1. 获取 inst 的 MRO 列表
2. 查找 cls 在当前 MRO 列表中的 index, 并返回它的下一个类，即 mro[index + 1]

当你调用 super(cls, inst) 时，Python 会在 inst 的 MRO 列表上搜索 cls 的下一个类。下面就详细分析一下:
首先看类 C 的` __init__ `方法：`super(C, self).__init__()`. 
这里的 self 是当前 C 的实例，self.class.mro() 结果是：
`[__main__.C, __main__.A, __main__.B, __main__.Base, object]`
可以看到，C 的下一个类是 A，于是，跳到了 A 的` __init__`，这时会打印出 enter A，并执行下面一行代码：`super(A, self).__init__()`, 
**注意:** 这里的 self 也是当前 C 的实例，MRO 列表跟上面是一样的，搜索 A 在 MRO 中的下一个类，发现是 B，于是，跳到了 B 的` __init__`，这时会打印出 enter B，而不是 enter Base。打印出enter B后, 搜索 B 在 MRO 中的下一个类是Base, 调用Base的`__init__`, 打印出enter Base 和 leave Base, 然后又跳到B的`__init__`,执行完 B 的
`__init__`, 再然后是 A 的`__init__`, 最后是 C 的`__init__`. 整个过程还是比较清晰的，关键是要理解 super 的工作方式，而不是想当然地认为 super 调用了父类的方法。

总结:
1. super 和父类没有实质性的关联。
2. super(cls, inst) 获得的是 cls 在 inst 的 MRO 列表中的下一个类。


### 参考
* [你不知道的super](https://funhacks.net/2016/11/09/super/)