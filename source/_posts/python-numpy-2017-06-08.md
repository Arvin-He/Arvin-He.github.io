---
title: Python之Numpy使用
date: 2017-06-08 09:34:41
tags: Python
categories: 编程
---
### Numpy相关介绍
NumPy 是一个 Python 包。 它代表 “Numeric Python”。 它是一个由多维数组对象和用于处理数组的例程集合组成的库。
使用NumPy，开发人员可以执行以下操作：
* 数组的算数和逻辑运算。
* 傅立叶变换和用于图形操作的例程。
* 与线性代数有关的操作, NumPy 拥有线性代数和随机数生成的内置函数。

NumPy 通常与 SciPy（Scientific Python）和 Matplotlib（绘图库）一起使用。 这种组合广泛用于替代 MatLab，是一个流行的技术计算平台, 此外NumPy 是开源的.

### Numpy安装
在命令行窗口,输入:`pip install numpy`, 回车后就自动安装.

### NumPy 之 Ndarray 对象 
NumPy 中定义的最重要的对象是称为 ndarray 的 N 维数组类型。 
它描述相同类型的元素集合。 可以使用基于零的索引访问集合中的项目。
基本的ndarray是使用 NumPy 中的数组函数创建的, ndarray 对象由计算机内存中的一维连续区域组成，带有将每个元素映射到内存块中某个位置的索引方案。 内存块以按行（C 风格）或按列（FORTRAN 或 MatLab 风格）的方式保存元素。

`numpy.array(object, dtype = None, copy = True, order = None, subok = False, ndmin = 0)`

```
object 任何暴露数组接口方法的对象都会返回一个数组或任何（嵌套）序列。
dtype 数组的所需数据类型，可选。
copy 可选，默认为true，对象是否被复制。
order C（按行）、F（按列）或A（任意，默认）。
subok 默认情况下，返回的数组被强制为基类数组。 如果为true，则返回子类。
ndimin 指定返回数组的最小维数。
```

```python
# 最小维度  
import numpy as np 
a = np.array([1,  2,  3,4,5], ndmin =  2)
b = np.array([1,  2,  3], dtype = complex)  
print(a)
print(b)

[[1, 2, 3, 4, 5]]
[ 1.+0.j,  2.+0.j,  3.+0.j]
```

### NumPy的数据类型
NumPy 支持比 Python 更多种类的数值类型
```
bool_ 存储为一个字节的布尔值（真或假）
int_ 默认整数，相当于 C 的long，通常为int32或int64
intc 相当于 C 的int，通常为int32或int64
intp 用于索引的整数，相当于 C 的size_t，通常为int32或int64
int8 字节（-128 ~ 127）
int16 16 位整数（-32768 ~ 32767）
int32 32 位整数（-2147483648 ~ 2147483647）
int64 64 位整数（-9223372036854775808 ~ 9223372036854775807）
uint8 8 位无符号整数（0 ~ 255）
uint16 16 位无符号整数（0 ~ 65535）
uint32 32 位无符号整数（0 ~ 4294967295）
uint64 64 位无符号整数（0 ~ 18446744073709551615）
float_ float64的简写
float16 半精度浮点：符号位，5 位指数，10 位尾数
float32 单精度浮点：符号位，8 位指数，23 位尾数
float64 双精度浮点：符号位，11 位指数，52 位尾数
complex_ complex128的简写
complex64 复数，由两个 32 位浮点表示（实部和虚部）
complex128 复数，由两个 64 位浮点表示（实部和虚部）
```

### 数据类型对象 (dtype)
数据类型对象描述了对应于数组的固定内存块的解释，取决于以下方面：
* 数据类型（整数、浮点或者 Python 对象）
* 数据大小
* 字节序（小端或大端）
* 在结构化类型的情况下，字段的名称，每个字段的数据类型，和每个字段占用的内存块部分。

```
numpy.dtype(object, align, copy)
Object：被转换为数据类型的对象。
Align：如果为true，则向字段添加间隔，使其类似 C 的结构体。
Copy ? 生成dtype对象的新副本，如果为flase，结果是内建数据类型对象的引用。
```

```python
# 使用数组标量类型  
#int8，int16，int32，int64 可替换为等价的字符串 'i1'，'i2'，'i4'，以及其他。  
import numpy as np 
dt = np.dtype(np.int32)  
dt2 = np.dtype('i4') 
# 使用端号标记
dt3 = np.dtype('>i4')  
print(dt)
print(dt2)
print(dt3)

int32
int32
>i4
```

```python
# 首先创建结构化数据类型。  
import numpy as np 
dt = np.dtype([('age',np.int8)])
# 将其应用于 ndarray 对象  
a = np.array([(10,),(20,),(30,)], dtype = dt) 

student = np.dtype([('name','S20'),  ('age',  'i1'),  ('marks',  'f4')]) 
stu = np.array([('abc',  21,  50),('xyz',  18,  75)], dtype = student)  

print(dt)
print(a)
# 名称可用于访问 age 列的内容  
print(a['age'])
print(stu)

# 输出结果
[('age', 'i1')]
[(10,) (20,) (30,)]
[10 20 30]
[('abc', 21, 50.0), ('xyz', 18, 75.0)]
```

每个内建类型都有一个唯一定义它的字符代码：
'b'：布尔值
'i'：符号整数
'u'：无符号整数
'f'：浮点
'c'：复数浮点
'm'：时间间隔
'M'：日期时间
'O'：Python 对象
'S', 'a'：字节串
'U'：Unicode
'V'：原始数据（void）

### NumPy - 数组属性
ndarray.shape : 返回一个包含数组维度的元组，可以用于调整数组大小
```python
import numpy as np 
a = np.array([[1,2,3],[4,5,6]])  

b = np.array([[1,2,3],[4,5,6]]) 
# 调整数组大小 
b.shape = (3,2)
c = a.reshape(3,2)
print(a.shape)
print(b)
print(c)
# 输出
(2, 3)

[[1, 2] 
 [3, 4] 
 [5, 6]]

[[1, 2] 
 [3, 4] 
 [5, 6]]  
```

ndarray.ndim
返回数组的维数。
numpy.itemsize
返回数组中每个元素的字节单位长度。

```python
In [1]: import numpy as np
In [5]: b = np.arange(24)                                                
                                                                         
In [6]: print(b)                                                         
[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23]
                                                                         
In [7]: c = b.reshape(2, 4, 3)                                           
                                                                         
In [8]: print(c)                                                         
[[[ 0  1  2]                                                             
  [ 3  4  5]                                                             
  [ 6  7  8]                                                             
  [ 9 10 11]]                                                            
                                                                         
 [[12 13 14]                                                             
  [15 16 17]                                                             
  [18 19 20]                                                             
  [21 22 23]]]                                                           
                                                                         
In [9]: print(c.ndim)                                                    
3    

In [10]: x = np.array([1,2,3,4,5], dtype = np.int8)

In [11]: print(x.itemsize)
1                                                                    
```

numpy.flags: 返回了它们的当前标志
1.  C_CONTIGUOUS (C) 数组位于单一的、C 风格的连续区段内
2.	F_CONTIGUOUS (F) 数组位于单一的、Fortran 风格的连续区段内
3.	OWNDATA (O) 数组的内存从其它对象处借用
4.	WRITEABLE (W) 数据区域可写入。 将它设置为flase会锁定数据，使其只读
5.	ALIGNED (A) 数据和任何元素会为硬件适当对齐
6.	UPDATEIFCOPY (U) 这个数组是另一数组的副本。当这个数组释放时，源数组会由这个数组中的元素更新

### NumPy - 数组创建例程
numpy.empty 
它创建指定形状和dtype的未初始化数组。 它使用以下构造函数：`numpy.empty(shape, dtype = float, order = 'C')`
**注意：**数组元素为随机值，因为它们未初始化。
numpy.zeros  
返回特定大小，以 0 填充的新数组。 `numpy.zeros(shape, dtype = float, order = 'C')`
numpy.ones   
返回特定大小，以 1 填充的新数组。 `numpy.ones(shape, dtype = None, order = 'C')`
```python
In [1]: import numpy as np

In [2]: x = np.empty([3,2], dtype=int)

In [3]: print(x)
[[64618546 64654448]
 [64655120 64654288]
 [64654960 64654864]]

In [4]: y = np.zeros(5)

In [5]: print(y)
[ 0.  0.  0.  0.  0.]

# 自定义类型
In [6]: a = np.zeros((2,2), dtype =  [('x',  'i4'),  ('y',  'i4')])

In [7]: print(a)
[[(0, 0) (0, 0)]
 [(0, 0) (0, 0)]]

In [8]: b = np.ones(5)

In [9]: print(b)
[ 1.  1.  1.  1.  1.]
```

### NumPy来自现有数据的数组
numpy.asarray 
将 Python 序列转换为ndarray。 `numpy.asarray(a, dtype = None, order = None)` 
其中: a 任意形式的输入参数，比如列表、列表的元组、元组、元组的元组、元组的列表
numpy.frombuffer
此函数将缓冲区解释为一维数组。 暴露缓冲区接口的任何对象都用作参数来返回ndarray。`numpy.frombuffer(buffer, dtype = float, count = -1, offset = 0) `
1.  buffer 任何暴露缓冲区接口的对象
2.	dtype 返回数组的数据类型，默认为float
3.	count 需要读取的数据数量，默认为-1，读取所有数据
4.	offset 需要读取的起始位置，默认为0

numpy.fromiter
从任何可迭代对象构建一个ndarray对象，返回一个新的一维数组。numpy.fromiter(iterable, dtype, count = -1)
```python
In [12]: s = 'Hello World'

In [13]: c = np.frombuffer(s, dtype='S1')
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-13-e77d207ef97e> in <module>()
----> 1 c = np.frombuffer(s, dtype='S1')

AttributeError: 'str' object has no attribute '__buffer__'

In [15]: c = np.fromiter(s, dtype='S1')

In [16]: print(c)
[b'H' b'e' b'l' b'l' b'o' b' ' b'W' b'o' b'r' b'l' b'd']
```

### NumPy来自数值范围的数组
numpy.arange
返回ndarray对象，包含给定范围内的等间隔值。`numpy.arange(start, stop, step, dtype)`
numpy.linspace
指定了范围之间的均匀间隔数量，而不是步长. `numpy.linspace(start, stop, num, endpoint, retstep, dtype)`
1.  start 序列的起始值
2.	stop 序列的终止值，如果endpoint为true，该值包含于序列中
3.	num 要生成的等间隔样例数量，默认为50
4.	endpoint 序列中是否包含stop值，默认为ture
5.	retstep 如果为true，返回样例，以及连续数字之间的步长
6.	dtype 输出ndarray的数据类型

numpy.logspace
返回一个ndarray对象，其中包含在对数刻度上均匀分布的数字。 刻度的开始和结束端点是某个底数的幂，通常为 10。`numpy.logscale(start, stop, num, endpoint, base, dtype)`
1.	start 起始值是base ** start
2.	stop 终止值是base ** stop
3.	num 范围内的数值数量，默认为50
4.	endpoint 如果为true，终止值包含在输出数组当中
5.	base 对数空间的底数，默认为10
6.	dtype 输出数组的数据类型，如果没有提供，则取决于其它参数

```python
In [17]: d = np.linspace(10, 20, 5)
In [18]: print(d)
[ 10.   12.5  15.   17.5  20. ]

In [19]: e = np.linspace(10, 20, 5, endpoint=False)
In [20]: print(e)
[ 10.  12.  14.  16.  18.]

In [22]: f = np.linspace(10, 20, 5, retstep=True)
In [23]: print(f)
(array([ 10. ,  12.5,  15. ,  17.5,  20. ]), 2.5)

# 默认底数是 10
In [25]: g = np.logspace(1.0, 2.0, num=10)
In [26]: print(g)
[  10.           12.91549665   16.68100537   21.5443469    27.82559402
   35.93813664   46.41588834   59.94842503   77.42636827  100.        ]

# 将对数空间的底数设置为 2
In [29]: a = np.logspace(1, 6, num=10, base=2)
In [30]: print(a)
[  2.           2.93946898   4.32023896   6.34960421   9.33223232
  13.71590373  20.1587368   29.62799079  43.54528001  64.        ]
```

### NumPy - 切片和索引
单维数组切片和list的切片一样, 下面主要介绍多维数组的切片和索引
切片还可以使用省略号（...），来使选择元组的长度与数组的维度相同。 如果在行位置使用省略号，它将返回包含行中元素的ndarray。
```python
In [31]: b = np.array([[1,2,3], [4,5,6], [7, 8, 9]])

In [32]: print(b[1:])
[[4 5 6]
 [7 8 9]]

# 返回第二列元素的数组
In [33]: print(b[..., 1])
[2 5 8]
# 返回第二行元素的数组
In [34]: print(b[1, ...])
[4 5 6]
# 返回从第二列向后切片所有元素：
In [36]: print(b[..., 1:])
[[2 3]
 [5 6]
 [8 9]]
```

### Numpy高级索引




### 参考
* [TutorialsPoint NumPy 教程](http://blog.csdn.net/wizardforcel/article/details/53932970)