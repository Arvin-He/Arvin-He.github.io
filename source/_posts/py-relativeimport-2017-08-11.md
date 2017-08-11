---
title: Python模块和包导入机制
date: 2017-08-11 15:53:58
tags: Python
categories: 编程
---
### 将代码封装成包
在文件系统上组织你的代码，并确保每个目录都定义了一个`__init__.py`文件,

### 关于相对导入
当导入模块时,报错:python systemerror parent module '' not loaded cannot perform relative import,这就是相对导入的问题了.

涉及到相对导入时，package所对应的文件夹必须正确的被python解释器视作package，而不是普通文件夹。
否则由于不被视作package，无法利用package之间的嵌套关系实现python中包的相对导入。

文件夹被python解释器视作package需要满足两个条件：
1. 文件夹中必须有`__init__.py`文件，该文件可以为空，但必须存在该文件。
2. 不能作为顶层模块来执行该文件夹中的py文件（即不能作为主函数的入口）。

补充：在"from YY import XX"这样的代码中，无论是XX还是YY，只要被python解释器视作package，就会首先调用该package的`__init__.py`文件。
如果都是package，则调用顺序是YY，XX。

### 一个问题
```python
test
|-- a.py
|-- b.py
`-- __init__.py

# a.py
from test.b import c
print(c)
# b.py
c = "test"

#运行 a.py
python3 a.py

Traceback (most recent call last):
File "a.py", line 1, in from test.b import c
ImportError: No module named 'test.b'
```

test 的上级目录不在 sys.path 中。
还有，不要直接执行一个包里边的文件。如果真需要执行一个包里的模块（而又不使用 distribute 提供的 entry point 安装配置），
请使用 `python3 -m test.a` 这样子。

### 将文件夹加入到 sys.path
你无法导入你的 Python 代码因为它所在的目录不在 sys.path 里。你想将添加新目录到 Python 路径，但是不想硬链接到你的代码。
有两种常用的方式将新目录添加到 sys.path:
1. 使用 PYTHONPATH环境变量来添加
2. 第二种方法是创建一个.pth 文件，将目录列举出来
```
# myapplication.pth
/some/dir
/other/dir
```
这个`.pth` 文件需要放在某个 Python 的 `site-packages` 目录，通常位于`/usr/local/lib/python3.3/site-packages` 或者 `˜/.local/lib/python3.3/sitepackages`。当解释器启动
时，`.pth` 文件里列举出来的存在于文件系统的目录将被添加到 `sys.path`。安装一个`.pth`文件可能需要管理员权限，如果它被添加到系统级的 Python 解释器。

### import的几种方式
1. 常规导入和重命名导入
```python
import os
import sys as system
```
2. 使用from语句导入
只想要导入一个模块或库中的某个部分:
`from functools import lru_cache`
从一个包中导入多个项：
```python
from os import path, walk, unlink
from os import uname, remove
```

上面是通过多次从同一个模块中导入实现的。当然，你也可以使用圆括号一次性导入多个项
```python
from os import (path, walk, unlink, uname, 
                remove, rename)
```                

也可以这样
```python
from os import path, walk, unlink, uname, \
                remove, rename
```

3. 相对导入
```
my_package/
    __init__.py
    subpackage1/
        __init__.py
        module_x.py
        module_y.py
    subpackage2/
        __init__.py
        module_z.py
    module_a.py
```

在顶层的`__init__.py`文件中，输入以下代码：                    
```python
from . import subpackage1
from . import subpackage2
```

然后进入subpackage1文件夹，编辑其中的`__init__.py`文件，输入以下代码：
```python
from . import module_x
from . import module_y
```

编辑`module_x.py`文件，输入以下代码：
```python
from .module_y import spam as ham

def main():
    ham()
```

编辑module_y.py文件，输入以下代码：
```python
def spam():
    print('spam ' * 3)
```

打开终端，cd至`my_package`包所在的文件夹，但**不要进入**my_package。在这个文件夹下运行Python解释器。
```python
In [1]: import my_package

In [2]: my_package.subpackage1.module_x
Out[2]: 

In [3]: my_package.subpackage1.module_x.main()
spam spam spam
```

**注意:** 如果你想要跨越多个文件层级进行导入，只需要使用多个句点即可。不过，PEP 328建议**相对导入的层级不要超过两层**。
还要注意一点，如果你往module_x.py文件中添加了`if __name__ == '__main__':`，然后试图运行这个文件，
你会碰到一个很难理解的错误。编辑一下文件，试试看吧！
```python
from . module_y import spam as ham

def main():
    ham()

if __name__ == '__main__':
    # This won't work!
    main()
```

从终端进入subpackage1文件夹，执行以下命令：
使用的是Python 2，你应该会看到下面的错误信息：
```
Traceback (most recent call last):
  File "module_x.py", line 1, in 
    from . module_y import spam as ham
ValueError: Attempted relative import in non-package
```

使用的是Python 3，错误信息是这样的：
```
Traceback (most recent call last):
  File "module_x.py", line 1, in 
    from . module_y import spam as ham
SystemError: Parent module '' not loaded, cannot perform relative import
```

这指的是，module_x.py是某个包中的一个模块，而你试图以脚本模式执行，但是这种模式不支持相对导入。
如果你想在自己的代码中使用这个模块，那么你必须将其添加至Python的导入检索路径（import search path）。最简单的做法如下：
```python
import sys
sys.path.append('/path/to/folder/containing/my_package')
import my_package
```

注意，你需要添加的是`my_package`的上一层文件夹路径，而不是`my_package`本身。原因是`my_package`就是我们想要使用的包，
所以如果你添加它的路径，那么将无法使用这个包。

4. 可选导入（Optional imports）
希望优先使用某个模块或包，但是同时也想在没有这个模块或包的情况下有备选，你就可以使用可选导入这种方式。
正如下面示例所示，可选导入的使用很常见，是一个值得掌握的技巧。这样做可以导入支持某个软件的多种版本或者实现性能提升。以github2包中的代码为例：
```python
try:
    # For Python 3
    from http.client import responses
except ImportError:  # For Python 2.5-2.7
    try:
        from httplib import responses  # NOQA
    except ImportError:  # For Python 2.4
        from BaseHTTPServer import BaseHTTPRequestHandler as _BHRH
        responses = dict([(k, v[0]) for k, v in _BHRH.responses.items()])
```

lxml包也有使用可选导入方式：
```python
try:
    from urlparse import urljoin
    from urllib2 import urlopen
except ImportError:
    # Python 3
    from urllib.parse import urljoin
    from urllib.request import urlopen
```

5. 局部导入
当你在局部作用域中导入模块时，你执行的就是局部导入。如果你在Python脚本文件的顶部导入一个模块，那么你就是在将该模块导入至全局作用域，这意味着之后的任何函数或方法都可能访问该模块。
```python
import sys  # global scope

def square_root(a):
    # This import is into the square_root functions local scope
    import math
    return math.sqrt(a)

def my_pow(base_num, power):
    return math.pow(base_num, power)

if __name__ == '__main__':
    print(square_root(49))
    print(my_pow(2, 3))
```

我们将sys模块导入至全局作用域，但我们并没有使用这个模块。
然后，在`square_root`函数中，我们将math模块导入至该函数的局部作用域，这意味着math模块只能在`square_root`函数内部使用。
如果我们试图在`my_pow`函数中使用math，会引发NameError.

使用局部作用域的好处之一，是你使用的模块可能需要很长时间才能导入，如果是这样的话，将其放在某个不经常调用的函数中或许更加合理，而不是直接在全局作用域中导入。老实说，我几乎从没有使用过局部导入，主要是因为如果模块内部到处都有导入语句，会很难分辨出这样做的原因和用途。**根据约定，所有的导入语句都应该位于模块的顶部**。
或者有的函数你只需使用一次可以选择局部导入.

### 导入注意事项
在导入模块方面，有几个程序员常犯的错误。这里介绍两个:
* 循环导入（circular imports）
* 覆盖导入（Shadowed imports，暂时翻译为覆盖导入）

#### 循环导入
如果你创建两个模块，二者相互导入对方，那么就会出现循环导入。例如：
```python
# a.py
import b

def a_test():
    print("in a_test")
    b.b_test()

a_test()
```

然后在同个文件夹中创建另一个模块，将其命名为b.py。
```python
import a

def b_test():
    print('In test_b"')
    a.a_test()

b_test()
```

如果你运行任意一个模块，都会引发AttributeError。这是因为这两个模块都在试图导入对方。简单来说，模块a想要导入模块b，但是因为模块b也在试图导入模块a（这时正在执行），模块a将无法完成模块b的导入。一般来说，**你应该做的是重构代码，避免发生这种情况**。

#### 覆盖导入
当你创建的模块与标准库中的模块同名时，如果你导入这个模块，就会出现覆盖导入。举个例子，创建一个名叫math.py的文件，在其中写入如下代码：
```python
import math

def square_root(number):
    return math.sqrt(number)

square_root(72)
```

现在打开终端，试着运行这个文件，你会得到以下回溯信息（traceback）：
```python
Traceback (most recent call last):
  File "math.py", line 1, in 
    import math
  File "/Users/michael/Desktop/math.py", line 6, in 
    square_root(72)
  File "/Users/michael/Desktop/math.py", line 4, in square_root
    return math.sqrt(number)
AttributeError: module 'math' has no attribute 'sqrt'
```

这到底是怎么回事？其实，你运行这个文件的时候，Python解释器首先在当前运行脚本所处的的文件夹中查找名叫math的模块。在这个例子中，解释器找到了我们正在执行的模块，试图导入它。但是我们的模块中并没有叫sqrt的函数或属性，所以就抛出了AttributeError。

### 参考
* [https://juejin.im/entry/570c6b6771cfe40067310370](https://juejin.im/entry/570c6b6771cfe40067310370)
* [cookbook]