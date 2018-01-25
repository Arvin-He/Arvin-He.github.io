---
title: Python之Monkeypatch
date: 2017-05-27 14:21:16
tags: python
categories: python
---
### 猴子补丁的由来
所谓的猴子补丁的含义是指在动态语言中，不去改变源码而对功能进行追加和变更。
猴子补丁的这个叫法起源于Zope框架，大家在修正Zope的Bug的时候经常在程序后面追加更新部分，这些被称作是“杂牌军补丁(guerilla patch)”，后来guerilla就渐渐的写成了gorllia(猩猩)，再后来就写了monkey(猴子)，所以猴子补丁的叫法是这么莫名其妙的得来的。

### 从Gevent学习猴子补丁的设计
猴子补丁这种东西充分利用了动态语言的灵活性，可以对现有的语言Api进行追加，替换，修改Bug，甚至性能优化等等。比如gevent的猴子补丁就可以对ssl、socket、os、time、select、thread、subprocess、sys等模块的功能进行了增强和替换。

### gevent中的猴子补丁模块gevent.monkey的设计和实现
如果自己要设计实现猴子补丁，可以按照这么个模式去做，可以用ipython来阅读python模块的代码，执行import gevent.monkey之后，只需要输入??gevent.monkey就可以查看源码了。

这个模块核心的函数其实就几个，分别是`get_original、patch_item、remove_item、patch_module`,还有一个全局变量叫做saved，默认指向一个空的字典对象。

首先来看patch_item函数：
这个函数的功能就是从指定模块中查找旧的项，并把旧的项保存到saved字典中，然后将旧项替换成新项。这里没有使用None，而是构建了一个空的object()作为默认属性.
```python
def patch_item(module, attr, newitem):
    NONE = object()
    olditem = getattr(module, attr, NONE)
    if olditem is not NONE:
        saved.setdefault(module.__name__, {}).setdefault(attr, olditem)
    setattr(module, attr, newitem)
```

patch_module的实现:
```python
def patch_module(name, items=None):
	gevent_module = getattr(__import__('gevent.' + name), name)
	module_name = getattr(gevent_module, '__target__', name)
	module = __import__(module_name)
	if items is None:
		items = getattr(gevent_module, '__implements__', None)
		if items is None:
			raise AttributeError('%r does not have __implements__' % gevent_module)
	for attr in items:
		patch_item(module, attr, getattr(gevent_module, attr))
```
gevent有个约定，作为补丁的gevent模块要包含这两个属性，`__target__和__implements__`，`__target__`是被补丁的默认模块名称，可以不指定，默认为gevent子模块的名称，比如gevent.socket是socket模块的补丁，
`__implements__`是要进行补丁的属性，这是gevent.socket模块中`__implements__`的定义：
```python
# standard functions and classes that this module re-implements in a gevent-aware way:
__implements__ = ['create_connection',
                  'socket',
                  'SocketType',
                  'fromfd',
                  'socketpair']
```
`patch_module`的工作就是从gevent模块里面读取这两个属性，然后遍历调用`patch_item`进行替换。

如果不希望用补丁的东西，而是使用原先的模块去进行处理，该怎么办？
前面提到过进行`patch_item`的时候会把旧的属性保存到名为saved的全局字典里面，如果要获得旧的模块属性，那么就要调用`get_original`函数从saved字典里面取出来。

### 猴子补丁使用建议

猴子补丁的功能很强大，但是也带来了很多的风险，尤其是像gevent这种直接进行API替换的补丁，整个Python进程所使用的模块都会被替换，可能自己的代码能hold住，但是其它第三方库，有时候问题并不好排查，即使排查出来也是很棘手，所以，就像松本建议的那样，如果要使用猴子补丁，那么**只是做功能追加，尽量避免大规模的API覆盖**。

### 参考
* [http://www.tuicool.com/articles/ema22iM](http://www.tuicool.com/articles/ema22iM)