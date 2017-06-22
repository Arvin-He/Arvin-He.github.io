---
title: 设计模式之单例模式
date: 2017-06-16 17:21:49
tags: [设计模式]
categories: 编程
---
### 单例模式
单例模式（Singleton Pattern）是一种常用的软件设计模式，该模式的主要目的是确保某一个类只有一个实例存在。
单例模式的要点有三个：
1. 某个类只能有一个实例；
2. 它必须自行创建这个实例；
3. 它必须自行向整个系统提供这个实例。
单例模式是一种对象创建型模式。单例模式又名单件模式或单态模式。

### python中的单例模式
在python中可以有多种方法实现单例模式:
* 使用模块
* 使用`__new__`
* 使用装饰器(decorator)
* 使用元类

#### 使用模块
其实，Python 的模块就是天然的单例模式，因为模块在第一次导入时，会生成 `.pyc` 文件，当第二次导入时，就会直接加载 `.pyc` 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做：
```python
# mysingleton.py
class My_Singleton(object):
    def foo(self):
        pass

my_singleton = My_Singleton()
```

将上面的代码保存在文件 mysingleton.py 中，然后这样使用：
```python
# 从mysingleton模块导入实例my_singleton
from mysingleton import my_singleton

my_singleton.foo()
```

#### 使用`__new__`
为了使类只能出现一个实例，我们可以使用 `__new__` 来控制实例的创建过程，代码如下：
```python
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kw):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kw)
        return cls._instance

class MyClass(Singleton):
    a = 1
```

在上面的代码中，我们将类的实例和一个类变量 `_instance` 关联起来，如果 `cls._instance` 为 None 则创建实例，否则直接返回 `cls._instance`。
执行情况如下:
```python
In [4]: one = MyClass()

In [5]: two = MyClass()

In [6]: one == two
Out[6]: True

In [7]: id(one)
Out[7]: 64402928

In [8]: id(two)
Out[8]: 64402928
```

#### 使用装饰器
装饰器（decorator）可以动态地修改一个类或函数的功能。这里，我们也可以使用装饰器来装饰某个类，使其只能生成一个实例，代码如下：
```python
from functools import wraps

def singleton(cls):
    instances = {}
    @wraps(cls)
    def getinstance(*args, **kw):
        if cls not in instances:
            instances[cls] = cls(*args, **kw)
        return instances[cls]
    return getinstance

@singleton
class MyClass(object):
    a = 1
```

在上面，我们定义了一个装饰器 singleton，它返回了一个内部函数 getinstance，该函数会判断某个类是否在字典 instances 中，如果不存在，则会将 cls 作为 key，cls(*args, **kw) 作为 value 存到 instances 中，否则，直接返回 instances[cls]。

#### 使用metaclass
元类（metaclass）可以控制类的创建过程，它主要做三件事：
* 拦截类的创建
* 修改类的定义
* 返回修改后的类
使用元类实现单例模式,代码如下:
```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call(*args, **kwargs)
        return cls._instance[cls]

# py2
#class MyClass(object):
#    __metaclass__ = Singleton

# py3
class MyClass(metaclass=Singleton):
    pass
```

以上是python中实现单例模式的一些方法

### C++中的单例模式
代码分析:
```cpp
// main.cpp
#include <iostream>
#include "Singleton.h"
using namespace std;

int main(int argc, char *argv[])
{
	Singleton * sg = Singleton::getInstance();
	sg->singletonOperation();
	
	return 0;
}
```

```cpp
// singleton.h
#ifndef SINGLETON_H
#define SINGLETON_H

class Singleton
{
public:
    virtual ~Singleton();
    // 提供一个公有的静态工厂方法
    static Singleton* getinstance();
    void SingletonOperation();

private:
    // 提供一个自身的静态私有成员变量
    static Singleton *instance;
    // 构造函数私有化
    Singleton();
};

#endif // SINGLETON_H
```

```cpp
// singleton.cpp
#include "singleton.h"
#include <iostream>
using namespace std;

Singleton *Singleton::instance = NULL;

Singleton::Singleton()
{

}

Singleton::~Singleton()
{
    delete instance;
}

Singleton* Singleton::getinstance()
{
    if (instance == NULL)
    {
        instance = new Singleton();
    }
    return instance;
}

void Singleton::SingletonOperation()
{
    cout<< "SingletonOperation" << endl;
}
```

运行结果:

![](design-patterns-2017-06-16/1.png)

#### 模式分析
单例模式的目的是保证一个类仅有一个实例，并提供一个访问它的全局访问点。单例模式包含的角色只有一个，就是单例类——Singleton。单例类拥有一个**私有构造函数**，确保用户无法通过new关键字直接实例化它。除此之外，该模式中包含一个**静态私有**成员变量与**静态公有**的工厂方法，该工厂方法负责检验实例的存在性并实例化自己，然后存储在静态成员变量中，以确保只有一个实例被创建。

在单例模式的实现过程中，需要注意如下三点：

* 单例类的构造函数为私有；
* 提供一个自身的静态私有成员变量；
* 提供一个公有的静态工厂方法。

### 优点
提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它，并为设计及开发团队提供了共享的概念。
由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象，单例模式无疑可以提高系统的性能。
允许可变数目的实例。我们可以基于单例模式进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例。

### 缺点
由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起。
滥用单例将带来一些负面问题，如为了节省资源将数据库连接池对象设计为单例类，可能会导致共享连接池对象的程序过多而出现连接池溢出；现在很多面向对象语言(如Java、C#)的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致对象状态的丢失。

### 适用环境
在以下情况下可以使用单例模式：

系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器，或者需要考虑资源消耗太大而只允许创建一个对象。
客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。
在一个系统中要求一个类只有一个实例时才应当使用单例模式。反过来，如果一个类可以有几个实例共存，就需要对单例模式进行改进，使之成为多例模式

### 参考
* [http://funhacks.net/2017/01/17/singleton/](http://funhacks.net/2017/01/17/singleton/)
* [图说设计模式](http://design-patterns.readthedocs.io/zh_CN/latest/index.html)