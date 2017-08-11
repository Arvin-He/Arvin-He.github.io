---
title: Python使用cx_freeze打包应用程序
date: 2017-08-04 13:20:52
tags: Python
categories: 编程
---

### python应用程序打包
python下应用程序打包有py2exe, pyinstaller和cx_freeze这3个第三方库,

目前python3.6不支持py2exe和pyinstaller,在win7 下 python3.6 使用 py2exe 或 pyinstaller 都报 Indexerror： tuple index out of range.

但pyinstaller和py2exe在python3.4版本可用.

### 使用cx_freeze打包应用程序
用cx_freeze打包出来的包文件很大,一个简单的程序打包出来大概有230M左右.
里面包括了python的runtime等各种依赖.

1. 安装cx_freeze
`pip install cx_freeze`

2. 创建setup.py脚本 
在你的工程的根目录下创建setup.py脚本,`setup.py`脚本可以是其他名字,如`setup_cx_freeze.py`,通常约定俗成是'setup.py'.只要不要和你的应用程序的脚本命令冲突就行.
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

3. 创建应用程序打包目录
打包的应用程序和依赖要放到一个文件夹中去, 这个文件夹必须在打包前创建好,
通常在应用程序根目录下新建一个build文件夹

4. 执行打包命令
执行打包命令: `python setup.py build`, 其中`setup.py`就是上面你写的脚本, build是应用程序被打包到的文件夹.

### 打包成*.msi格式
在命令行窗口输入:`python setup.py bdist_msi`,回车,就会在build目录下生成一个*.msi格式的软件安装包

### 制作exe安装包
windows下通常分发软件都是`*.exe`格式的,`*.msi`也可以,python的分发包就是`*.msi`格式的.

但是python打包出来的文件都在一个文件夹,并不是一个exe文件,而且图标, 配置文件并不会被打包进来.此外该文件夹的体积很大.

考虑到上面的种种情况, windows下需要将上面文件夹的文件再次压缩打包成一个exe, 比较好的工具是NSIS.

1. 安装NSIS
NSIS有window版和unicode版本, 注意添加相关环境变量

2. 编写打包脚本
创建一个打包脚本,如pack.nsi, 然后在该脚本中写脚本

3. 执行命令脚本打包
命令: `makensis pack.nsi`

### 制作deb包
linux下分发软件是deb包的形式


### 一些问题
使用cx_freeze打包python程序,在打包sqlalchemy程序时,`C:\programs File\Python36\Lib\site-packages\sqlalchemy\sql\default_comparator.pyc`这个模块没有被打包进来,但是其他模块都被打包进来了.
解决办法: 复制`default_comparator.py`文件或者在`__pychae__`目录下复制`default_comparator.pyc`到你的打包目录中对应的目录.然后再通过NSIS打包.

在python3.4中使用cx_freeze打包能将sqlalchemy中的`default_comparator.pyc`打包,在python3.6却唯独漏掉这个`default_comparator.pyc`,原因未知,
解决办法:
在`cx_freeze`的`setup.py`脚本中的`build_exe_options`中的packages中添加sqlalchemy,这样就会将sqlalchemy完成打包进来,不会漏掉一个模块,格式如下:
```python
# 依赖会自动检测,但会需要微调
build_exe_options = {
    "packages": ["sqlalchemy"],
    "excludes": ["tkinter"],
    "includes": [],
    "include_files": []
}
```


### 关于cx_freeze的setup.py的脚本问题
使用cx-freeze进行打包时,setup.py必须放在程序运行脚本的根目录下,如果将setup.py放到同级的一个文件夹中(如make文件夹,为了不污染源代码),打包出来后的exe运行报错.提示模块找不到.
