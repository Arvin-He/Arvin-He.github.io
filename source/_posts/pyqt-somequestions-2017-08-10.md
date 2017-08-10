---
title: PyQt5遇到的一些问题
date: 2017-08-10 13:09:10
tags: PyQt5
categories: 编程
---
### 关于QButtonGroup的问题
在循环中对按钮做属性修改,比如setchecked属性.
之前将所有的button对象放到QButtonGroup中,然后在循环中设置属性,代码如下:
```python
for btn in self.buttonGroup.buttons():
    btn.setCheckable(True)
    btn.setChecked(True)
```

最后结果是在groupbutton中的最后一个button会被设置为checked状态,其他的按钮都没有被设置为checked状态.

解决办法:将按钮对象放在一个list里,然后遍历list里的按钮对象就行了.后来查了文档,放在QButtonGroup中的按钮对象,在放入QButtonGroup之前必须具有SetCheckabled为true这个属性.否则无法设置setchecked这个属性.
