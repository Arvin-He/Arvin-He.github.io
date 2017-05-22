---
title: python之cx_Freeze使用
date: 2017-05-22 17:26:02
tags: Python
categories: 编程
---

### cx_Freeze使用
与py2exe一样,需要在要打包的程序目录下创建一个setup.py, 当然也可以是其他名字
然后在命令行差un港口输入:`python setup.py build`,回车,然后就会在程序目录下
创建一个build文件夹,里面包含了需要用到的依赖
```python
# setup.py
import sys
from cx_Freeze import setup, Executable
# 依赖关系被自动检测，但可能需要微调
build_exe_options = {"packages": ["os"], "excludes": ["tkinter"]}
# GUI应用程序在Windows上需要不同的基础（默认值为控制台应用程序)
base = None
if sys.platform == "win32":
    base = "Win32GUI"

setup(  name = "guifoo",
        version = "0.1",
        description = "My GUI application!",
        options = {"build_exe": build_exe_options},
        executables = [Executable("guifoo.py", base=base)])

```

### 打包成*.msi格式
在命令行窗口输入:`python setup.py bdist_msi`,回车,就会在build目录下生成一个*.msi格式的软件安装包


### 参考
* [http://cx-freeze.readthedocs.io/en/latest/distutils.html](http://cx-freeze.readthedocs.io/en/latest/distutils.html)