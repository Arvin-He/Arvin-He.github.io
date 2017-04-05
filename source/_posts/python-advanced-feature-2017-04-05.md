---
title: Python高级特性
date: 2017-04-05 20:06:51
tags: Python
categories: 编程
---
### 1. 列表生成式
```
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# for循环后面还可以加上if判断
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]

# 使用两层循环，生成全排列
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

### 2. 生成器
创建一个generator，有很多种方法。
第一种，只要把一个列表生成式的[]改成()，就创建了一个generator.
定义generator的另一种方法。如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个generator
变成generator的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行。