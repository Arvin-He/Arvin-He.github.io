---
title: Python之列表解析式
date: 2017-04-02 20:06:51
tags: python
categories: python
---
### 列表解析式概念
列表解析式是将一个列表（实际上适用于任何可迭代对象（iterable））转换成另一个列表的工具。
在转换过程中，可以指定元素必须符合一定的条件，才能添加至新的列表中，这样每个元素都可以按需要进行转换。
如果熟悉函数式编程（functional programming），可以把列表解析式看作为结合了filter函数与map函数功能的语法糖：

### 列表解析式应用
列表解析式可用来过滤列表中的元素,而不必使用循环加判断的方式

### 循环与解析式
每个列表解析式都可以重写为for循环，但不是每个for循环都能重写为列表解析式。

### 无条件子句的列表解析式
```python
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

### 嵌套循环
示例如下:
```python
# 使用两层循环，生成全排列
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

下面是一个拉平（flatten）矩阵（以列表为元素的列表）的for循环：
```python
flattened = []
for row in matrix:
    for n in row:
        flattened.append(n)
```
下面这个列表解析式实现了相同的功能：
`flattened = [n for row in matrix for n in row]`

如果要在列表解析式中处理嵌套循环，请记住for循环子句的顺序与我们原来for循环的顺序是一致的。
若写成`flattened = [n for n in row for row in matrix]`则是错误的.
同样地原则也适用集合解析式（set comprehension）和字典解析式（dictionary comprehension）。

### 带条件子句的列表解析式
列表生成式可用来过滤列表中的元素
```
# for循环后面还可以加上if判断
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```

### 字典解析式
```python
flipped = {value: key for key, value in original.items()}
```

### 注意可读性
Python支持在括号和花括号之间断行
```python
doubled_odds = [
    n * 2
    for n in numbers
    if n % 2 == 1
]

flattened = [
    n
    for row in matrix
    for n in row
]
flipped = {
    value: key
    for key, value in original.items()
}
```


### 参考文档
* [Python官方文档](https://docs.python.org/3/library/asyncio.html)
* [廖雪峰Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)
* [http://codingpy.com/article/python-list-comprehensions-explained-visually/](http://codingpy.com/article/python-list-comprehensions-explained-visually/)