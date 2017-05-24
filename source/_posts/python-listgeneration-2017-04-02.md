---
title: Python之列表生成器
date: 2017-04-02 20:06:51
tags: Python
categories: 编程
---
### 列表生成式
列表生成式可用来过滤列表中的元素
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




### 参考文档
* [Python官方文档](https://docs.python.org/3/library/asyncio.html)
* [廖雪峰Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)