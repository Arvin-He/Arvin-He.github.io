---
title: cookbook之C 语言扩展
date: 2017-07-06 08:28:17
tags: python
categories: python
---
### 使用C扩展
开发者有三种方法可以在自己的Python代码中来调用C编写的函数: ctypes，SWIG，Python/C API。每种方式也都有各自的利弊。
首先，我们要明确为什么要在Python中调用C？
常见原因如下：
1. 你要提升代码的运行速度，而且你知道C要比Python快50倍以上
2. C语言中有很多传统类库，而且有些正是你想要的，但你又不想用Python去重写它们
3. 想对从内存到文件接口这样的底层资源进行访问
4. ...

### 使用 ctypes 访问 C 代码
你有一些 C 函数已经被编译到共享库或 DLL 中。你希望可以使用纯 Python 代码调用这些函数，而不用编写额外的 C 代码或使用第三方扩展工具.
对于需要调用 C 代码的一些小的问题，通常使用 Python 标准库中的 ctypes 模块就足够了
```c
// sample.c
#include <math.h>

int my_add(int x, int y)
{
    return x + y;
}

int gcd(int x, int y)
{
    int g = y;
    while (x > 0)
    {
        g = x;
        x = y % x;
        y = g;
    }
    return g;
}

int in_mandel(double x0, double y0, int n){
    double x = 0, y = 0, xtemp;
    while (n > 0) {
        xtemp = x*x - y*y + x0;
        y = 2*x*y + y0;
        x = xtemp;
        n -= 1;
        if (x*x + y*y > 4) return 0;
    }
    return 1;
}

int divide(int a, int b, int *remainder){
    int quot = a / b;
    *remainder = a % b;
    return quot;
}

double avg(double  *a, int n) {
    int i;
    double total = 0.0;
    for (i = 0; i < n; i++) {
        total += a[i];
    }
    return total / n;
}

typedef struct Point{
    double x, y;
}Point;

double distance(Point *p1, Point *p2){
    return hypot(p1->x - p2->x, p1->y - p2->y);
}
```

然后将sample.c编译成libsample.so: `gcc -shared -o libsample.so -fPIC sample.c`
**注意:**对于 C 和 Python 代码一起打包的问题，如果你在使用 ctypes 来访问编译后的 C 代码，那么需要确保这个共享库放在sample.py 模块同一个地方.如果 C 函数库被安装到其他地方，那么你就要修改相应的路径。如果 C 函数库在你机器上被安装为一个标准库了，那么可以使用 `ctypes.util.find_library()` 函数来查找.
```python
>>> from ctypes.util import find_library
>>> find_library('sample')
'/usr/local/lib/libsample.so'
```

```python
# sample.py
import ctypes
import os

_file = "libsample.so"
_path = os.path.join(os.path.abspath("."), _file)
# _path = os.path.join(*(os.path.split(__file__)[:-1]+(_file,)))
_mod = ctypes.cdll.LoadLibrary(_path)


my_add = _mod.my_add
my_add.argtypes = (ctypes.c_int, ctypes.c_int)
my_add.restype = ctypes.c_int


gcd = _mod.gcd
gcd.argtypes = (ctypes.c_int, ctypes.c_int)
gcd.restype = ctypes.c_int


in_mandel = _mod.in_mandel
in_mandel.argtypes = (ctypes.c_double, ctypes.c_double, ctypes.c_int)
in_mandel.restype = ctypes.c_int


_divide = _mod.divide
_divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))
_divide.restype = ctypes.c_int

def divide(x, y):
    rem = ctypes.c_int()
    quot = _divide(x, y, rem)
    return quot, rem.value

class DoubleArrayType:
    def from_param(self, param):
        typename = type(param).__name__
        if hasattr(self, "from_" + typename):
            return getattr(self, "from_" + typename)
        elif isinstance(param, ctypes.Array):
            return param
        else:
            raise TypeError("can't convert %s" % typename)

    def from_array(self, param):
        if param.typecode != 'd':
            raise TypeError("must be an array of doubles")
        ptr, _ = param.buffer_info()
        return ctypes.cast(ptr, ctypes.POINTER(ctypes.c_double))

    def from_list(self, param):
        val = ((ctypes.c_double)*len(param))(*param)
        return val

    from_tuple = from_list

    def from_ndarray(self, param):
        return param.ctypes.data_as(ctypes.POINTER(ctypes.c_double))


DoubleArray = DoubleArrayType()
_avg = _mod.avg
_avg.argtypes = (DoubleArray, ctypes.c_int)
_avg.restype = ctypes.c_double


def avg(values):
    return _avg(values, len(values))

class Point(ctypes.Structure):
    _fields_ =[('x', ctypes.c_double),
               ('y', ctypes.c_double)]

distance = _mod.distance
distance.argtypes = (ctypes.POINTER(Point), ctypes.POINTER(Point))
distance.restype = ctypes.c_double
```

```python
# main.py
import sample

print(sample.gcd(35, 42))
print(sample.my_add(10, 20))
print(sample.in_mandel(0,0,500))
print(sample.divide(42, 8))
print(sample.avg([1,2,3]))

>>>
aron@aron-VirtualBox:~/Desktop/pycallc$ python3 main.py
7
30
1
(5, 2)
2.0
```

一旦你知道了C函数库的位置，那么就可以像下面这样使用`ctypes.cdll.LoadLibrary()` 来加载它，其中 `_path` 是标准库的全路径,加载模块`_mod = ctypes.cdll.LoadLibrary(_path)`,函数库被加载后，你需要编写几个语句来提取特定的符号并指定它们的类型
```python
# int in_mandel(double, double, int)
in_mandel = _mod.in_mandel
in_mandel.argtypes = (ctypes.c_double, ctypes.c_double, ctypes.c_int)
in_mandel.restype = ctypes.c_int
```

argtypes 属性是一个元组，包含了某个函数的输入参数类型
restype  是相应的返回类型
想让 Python 能够传递正确的参数类型并且正确的转换数据的话，那么这些类型签名的绑定是很重要的一步。如果你没有这么做，不但代码不能正常运行，还可能会导致整个解释器进程挂掉。使用
ctypes 有一个麻烦点的地方是原生的 C 代码使用的术语可能跟 Python 不能明确的对应上来。 divide() 函数是一个很好的例子，它通过一个参数除以另一个参数返回一个结果值。尽管这是一个很常见的 C 技术，但是在 Python 中却不知道怎样清晰的表达出来。

最后一些小的提示：如果你想在 Python 中访问一些小的 C 函数，那么 ctypes 是
一个很有用的函数库。尽管如此，如果你想要去访问一个很大的库，那么可能就需要
其他的方法了，比如 Swig 或 Cython.

由于 ctypes 并不是完全自动化，那么你就必须花费大量时间来编写所有的类型签名，
如果函数库够复杂，你还得去编写很多小的包装函数和支持类。
作为 ctypes 的一个替代，你还可以考虑下 CFFI。CFFI 提供了很多类似的功能，但是使用 C 语法并支持更多高级的 C 代码类型。

###  Python/C API
不依靠其他工具，直接使用 Python 的扩展 API 来编写一些简单的 C 扩展模块。
Python/C API可能是被最广泛使用的方法。它不仅简单，而且可以在C代码中操作你的Python对象。
这种方法需要**以特定的方式来编写C代码**以供Python去调用它。所有的Python对象都被表示为一种叫做PyObject的结构体，并且Python.h头文件中提供了各种操作它的函数。

```c
#include <Python.h>
 
static PyObject* add(PyObject* self, PyObject* args){
    int a = 0;
    int b = 0;
    if(!PyArg_ParseTuple(args, "i|i", &a, &b))
        return NULL;
    return Py_BuildValue("i", a+b);
}
 
static PyObject* sub(PyObject* self, PyObject* args){
    int a = 0;
    int b = 0;
    if(!PyArg_ParseTuple(args, "i|i", &a, &b))
        return NULL;
    return Py_BuildValue("i", a-b);
}
 
static PyMethodDef addMethods[]={
    {"add", add, METH_VARARGS},
    {"sub", sub, METH_VARARGS},
    {NULL, NULL, 0, NULL}
};

/* Module structure */
static struct PyModuleDef mytestmodule = {
    PyModuleDef_HEAD_INIT,
    "mytest", /* name of module */
    "A sample module", /* Doc string (may be NULL) */
    -1, /* Size of per-interpreter state or -1 */
    addMethods /* Method table */
};
 
PyMODINIT_FUNC PyInit_mytest(void) {
    return PyModule_Create(&mytestmodule);
} 
```

mytest.c编写好后就使用命令或者使用setup.py生成.so文件
方法1:使用命令生成.so
`gcc -shared -I /usr/include/python3.5 -o mytest.so -fPIC mytest.c `
方法2: 编写setup.py生成.so文件
```python
# setup.py
from distutils.core import setup, Extension

setup(name='mytest',
      ext_modules=[
      Extension('mytest',
                ['mytest.c'],
                include_dirs = ['/usr/include/python3.5'],
                )])
```

**注意:**使用setup.py生成的.so文件名字会变成`mytest.cpython-35m-x86_64-linux-gnu.so`,不是`mytest.so`,
重命名`mytest.cpython-35m-x86_64-linux-gnu.so`为`mytest.so`,不然会报找不到`Error:ImportError: No module named mytest`.

生成好.so文件,就直接在python代码中import,然后调用函数了
```python
import mytest

print(mytest.add(5, 5))
print(mytest.sub(20, 6))

<<< 10
<<< 14
```

### 从 C 语言中调用 Python 代码
想在 C 语言中使用某个 Python 函数作为一个回调。
在 C 语言中调用 Python，要记住最重要的是 C 语言会是主体。也就是说，C 语言负责构造参数、调用 Python 函数、检查异常、检查类型、提取返回值等.


### 用 WSIG 包装 C 代码
首先要说明的是swig可以进行很多语言的调用转换，不止是可以让python调用c。swig和sip都被称作wrapper，就是说他对你的原有函数进行了包装。看到之前用python c api的方式里，我们必须严格按照python c api的方式来写代码，破坏了原有c程序的可读性，于是wrapper的思想就是把原生c程序包装成python c api那种方式的代码，再去生成so文件。因此我们要做的是首先写c文件。
```c
int add(int a, int b){
    return a+b;
}
int sub(int a, int b){
    return a-b;
}
```

然后再去写一个swig格式的接口文件。
```
%module mytest
%{
extern int add(int a, int b);
extern int sub(int a, int b);
%}
 
extern int add(int a, int b);
extern int sub(int a, int b);
```

然后就可以运行swig，他会自动生成python c api写的代码，并且会自动编译出so文件来调用。


### 用 sip 包装 C 代码
sip是swig发展而来是方便python调用c的，所以基本使用方式都是差不多，只不过接口文件略有差异.


### cffi
cffi类似于ctypes直接在python程序中调用c程序，但是比ctypes更方便不要求编译成so再调用，
注意到上面的所有方式都是需要去编译成so文件后再在python中调用，而cffi允许你直接调用c文件来使用里面的函数了，为什么这么神奇呢，其实是cffi在解释过程中才帮你把c编译为so文件.
```python
from cffi import FFI
ffi = FFI()
ffi.cdef("""
int add(int a, int b);
int sub(int a, int b);
""")
lib = ffi.verify('#include "mytest.c"')
print(lib.add(1,2))
```