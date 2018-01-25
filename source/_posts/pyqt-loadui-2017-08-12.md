---
title: PyQt5加载ui的几种方式
date: 2017-08-12 10:52:06
tags: PyQt5
categories: python
---

### 前言
一般界面的创建有两种方式:
1. 使用纯代码实现
2. 使用designer来拖拽控件
两种方式各有各自的好处,视具体的使用场景来选择.

使用纯代码实现时一些属性需要在代码中指定, 或者继承某个控件类并定制添加一些属性和功能.这种方式比较灵活.但是控件太多且是不同种类的控件的话就有点繁琐了.

使用designer的方式是直观,快速,属性可视化,可添加动态属性,这也是使用比较多的方式.在designer中设计好界面后,保存为一个后缀为ui的文件,用文本编辑器打开是一个xml格式的文件,里面指明控件的各个属性.

下面就讲述在代码中加载ui的几种方式.

### PyQt5中加载ui的方式
PyQt5中加载ui的方式主要有3种:
1. 直接加载ui文件
2. 将ui文件转成py文件加载
3. 将所有的资源文件(包括ui,图片等)编译成内容是字节的py文件加载

### 直接加载ui文件
使用uic加载ui文件,看下面代码
```python
from PyQt5 import QtWidgets, uic, QtCore

class MyDialog(QtWidgets.QDialog):

    def __init__(self):
        super(MyDialog, self).__init__()
        uic.loadUi(os.path.join(os.path.dirname(__file__), "yourDialog.ui"), self)
        self.initUI()

```

ui的控件直接通过`self`来获取访问,比如`self.mylabel.setText("xxx")`,这是最直接的方式.
适用于简单的界面设计.

### 将ui文件转成py文件加载
Qt Designer默认继承的object类，但不提供show()显示方法.
如何将ui文件转为py文件?
使用pyqt5中自带的工具pyuic5,pyuic5是一个可执行文件,在控制台可作为命令使用,具体使用参考下面的脚本
```python
# -*- coding:utf-8 -*-
import os
import sys
import subprocess

inputFile = os.path.abspath(os.path.join("../res", "serialCom.ui"))
print("input file ={}".format(inputFile))
outputFile = os.path.abspath("../serialCom_ui.py")
print("output file ={}".format(outputFile))
try:
    # py3.4.3
    subprocess.call(["pyuic5.bat", inputFile, "-o", outputFile])
except:
    # py3.6
    subprocess.call(["pyuic5", inputFile, "-o", outputFile])
print("build ui done.")
```

根据上面的脚本会根据ui文件生成一个py文件,那么这个py文件有哪些内容呢?
```python
from PyQt5 import QtCore, QtGui, QtWidgets

class Ui_serialDlg(object):
    def setupUi(self, serialDlg):
        serialDlg.setObjectName("serialDlg")
        serialDlg.resize(1024, 768)
        sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Fixed, QtWidgets.QSizePolicy.Fixed)
        # 这里省略了ui控件的属性设定
        ...
        self.retranslateUi(serialDlg)
        QtCore.QMetaObject.connectSlotsByName(serialDlg)

    def retranslateUi(self, serialDlg):
        _translate = QtCore.QCoreApplication.translate
        # 这里略去了各种翻译的内容
        ...
```

从这个py看出,这个py文件生成了一个类,里面有2个函数,分别是`setupUi`和`retranslateUi`.
setupUi中主要是控件的各个属性设置, retranslateUi主要是翻译的内容
下面如何在你的代码中加载这个py文件呢?
有两种方式
第一种: 直接继承这个类, python中支持多继承
```python
from serialCom_ui import Ui_serialDlg as serialDlg

class MainWindow(QDialog, serialDlg):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.setupUi(self)
```

第二种: 在你的代码中实例化
```python
from serialCom_ui import Ui_serialDlg as serialDlg

class MainWindow(QDialog):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.serial_dlg=serialDlg()  
        self.serial_dlg.setupUi(self)  
```

为什么要生成py文件呢?主要是为了实现代码与界面分离。
缺点:ui文件有了更改,必须要再次生成对应的py文件,如果你忘记生成了,就会造成ui不同步.
优点:不再需要这个ui文件了.这就相当于使用纯代码的实现方式了.但这样的方式显然比纯代码要快,而且能做到
逻辑代码和界面代码分离.

### 生成qrc文件编译资源文件生成py文件加载
当有ui文件还有图片等资源文件时,怎么办呢?当图片重命名了怎么办? 当然是利用Qt的资源系统来整合这些资源文件了.
Qt 资源系统是一个跨平台的资源机制，用于将程序运行时所需要的资源以二进制的形式存储于可执行文件内部。
如果你的程序需要加载特定的资源（图标、文本翻译等），那么，将其放置在资源文件中，就再也不需要担心这些文件的丢失。
也就是说，如果你将资源以资源文件形式存储，它是会编译到可执行文件内部。
怎么做呢?
一般把用到的资源文件放到一个文件夹中,如res/,然后创建一个资源文件*.qrc,该文件生成在res文件夹中
qrc文件的内容有,如下:
```
<RCC><qresource prefix="..">
  <file mtime="1502414194.7463503">favor.ico</file>
  <file mtime="1494807198.7998164">favor.png</file>
  <file mtime="1502505175.56">serialCom.ui</file>
  <file mtime="1502414207.032053">uninst.ico</file>
</qresource></RCC>
```

python中创建qrc文件,然后通过pyrcc5将资源文件编译到py文件中去
这里不仅仅生成qrc文件,还对qrc文件记录了资源文件最后的修改时间,并做了修改时间对比,
一旦有ui文件被修改了,就会重新编译生成res_rc.py文件.
```python
# 自动编译加载资源
_res_path = os.path.abspath('res')
RCC = """<RCC><qresource prefix="{}">\n{}</qresource></RCC>"""
FILE = """  <file mtime="{}">{}</file>\n"""

def _loadRes(res, root):
    # res文件夹的路径
    package = os.path.dirname(res)
    # 生成资源清单数据
    res_files = []
    for a, _, files in os.walk(res):
        for f in files:
            if f == "res.qrc" or f.endswith(".ts"):
                continue
            ff = os.path.join(a, f)
            res_files.append((os.path.getmtime(ff), os.path.relpath(
                ff, res).replace(os.path.sep, "/")))

    res_qrc_data = RCC.format(
        os.path.relpath(package, root).replace(os.path.sep, "/"),
        "".join([FILE.format(*x) for x in res_files]))

    # 更新资源清单
    res_qrc = os.path.join(res, "res.qrc")
    res_updated = False

    # 检查现有资源清单是否已是最新
    if os.path.exists(res_qrc):
        with open(res_qrc, "r", encoding="utf-8") as f:
            res_updated = f.read() == res_qrc_data

    # 更新资源清单
    if not res_updated:
        with open(res_qrc, "w", encoding="utf-8") as f:
            f.write(res_qrc_data)

    # 编译资源清单
    res_rc_py = os.path.join(package, "res_rc.py")
    if not os.path.exists(res_rc_py) or \
            os.path.getmtime(res_rc_py) < os.path.getmtime(res_qrc):
        # 通过指定 `cwd` 解决 win32 下 pyrcc5 不支持中文路径的问题
        rel_res_rc_py = os.path.relpath(res_rc_py, package)
        rel_res_qrc = os.path.relpath(res_qrc, package)

        subprocess.check_call(
            ["pyrcc5", "-o", rel_res_rc_py, rel_res_qrc], cwd=package)

# 加载资源
for path, b, c in os.walk(_res_path):
    if path.endswith(os.path.sep + "res"):
        _loadRes(path, _res_path)
```

有了`res_rc.py`文件,然后将`res_rc.py`文件import进来,然后让问资源文件通过指定路径访问.
```python
from PyQt5 import QtCore

import res_rc
# :不能少, 路径就是qrc里的prefix+资源文件名
pixmap = QPixamp(":/prefix/download.jpeg")
```

或者
```python
def loadUi(widget, uiFileName):
    """加载 ui. 从 uiFileName 指定的 ui 文件加载."""
    f = QtCore.QFile(uiFileName)
    f.open(QtCore.QFile.ReadOnly | QtCore.QFile.Text)
    assert f.isOpen()
    ts = QtCore.QTextStream(f)
    ts.setCodec("utf-8")
    code_string = io.StringIO()
    winfo = uic.compiler.UICompiler().compileUi(ts, code_string, True, "_rc")
    ui_globals = {"__name__": widget.__module__}
    exec(code_string.getvalue(), ui_globals)
    Ui = ui_globals[winfo["uiclass"]]
    ui = Ui()
    ui.setupUi(widget)
    return ui

class MainWindow(QDialog, serialDlg):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.ui = loadUi(self, ":../serialCom.ui")
```

然后所有ui里的控件对象都可以使用self.ui来访问了.
