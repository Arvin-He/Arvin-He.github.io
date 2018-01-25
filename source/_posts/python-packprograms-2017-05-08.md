---
title: Python之应用程序打包发布
date: 2017-05-08 11:20:24
tags: python
categories: python
---

### Windows上Python程序打包发布方式
* .py文件：直接提供源码，使用者自行安装Python并且安装依赖的各种库。(Python官方的各种安装包就是这样做的)
* .pyc文件：不愿意公开源码,可使用pyc文件发布，pyc文件是Python解释器可以识别的二进制码，故发布后也是跨平台的，需要使用者安装相应版本的Python和依赖库。
* 可执行文件：包含了Python和依赖库,用户只要点击快捷方式即可. 比较麻烦的是需要针对不同平台需要打包不同的可执行文件.

### 各种打包工具的对比如下：

|Solution	|Windows	|Linux	|OS X	|Python 3	|License	|One-file mode	|Zipfile import	|Eggs	|pkg_resources support|
|:--------|---------|-------|-----|---------|---------|---------------|---------------|-----|---------------------|
|bbFreeze	  |yes	|yes	|yes	|no	  |MIT	|no	  |yes	|yes	|yes|
|py2exe	    |yes	|no	  |no	  |yes	|MIT	|yes	|yes	|no	  |no|
|pyInstaller|yes	|yes	|yes	|yes	|GPL	|yes	|no	  |yes	|no|
|cx_Freeze	|yes	|yes	|yes	|yes	|PSF	|no	  |yes	|yes	|no|
|py2app	    |no	  |no	  |yes	|yes	|MIT	|no	  |yes	|yes	|yes|

PS.其中pyInstaller和cx\_Freeze都是不错的，stackoverflow上也有人建议用cx\_Freeze，说是更便捷些。pkg\_resources新版的pyInstaller貌似是支持的。

### 使用py2exe打包发布程序

#### py2exe打包发布
**注意:** 目前py2exe只支持到3.4版本,3.6版本不支持,主要是语法上不支持.

1. 安装py2exe: `pip install py2exe`
2. 准备好你要打包的程序,如hello.py
```python
# hello.py
print("hello")
```
3. 创建安装脚本程序（setup.py）
使用py2exe工具里需要一个setup.py的脚本，在你需要打包的应用程序目录下创建一个setup.py脚本文件,具体内容如下：
```python
#python 3.4
from distutils.core import setup
import py2exe

setup(console=['hello.py'])
```
在这个脚本里调用setup函数，创建控制台应用程序，它的入口主文件是hello.py文件。

3. 运行脚本（setup.py）文件
在控制台窗口中进入到需要打包的应用程序目录下,在控制台窗口里输入命令：`python setup.py py2exe`,回车,然后就开始自动打包.
运行这个命令成功之后，会在当前的目录下面创建一个发布的目录dist，所有需要发布的文件就会拷贝到此目录下面。

4. 执行生成的exe程序
经过上面的步骤，就可以进入目录dist下面进行运行exe程序了,双击就可运行.运行成功之后，与前面使用python hello.py是一样的结果，不过这个目录内容就可以发布到不同的电脑上进行运行，且不需要安装python。

#### py2exe使用出现的问题
1. py2exe 在打包简单的文件时能够正常工作,但打包稍微复杂的应用程序时会出现递归溢出的问题,打包失败,不知原因,且网上没有好的解决方法.
2. 如果是PyQt的GUI应用程序,py2exe则不会将平台相关的东西(即PyQt下的platform文件夹)打包进来.

### 使用Pyinstaller打包发布程序

#### Pyinstaller安装
控制台窗口输入:`pip install pyinstaller`,回车,然后开始自动安装pyinstaller

#### Pyinstaller使用
目录切换到你要打包程序的目录下, 在控制台窗口输入:`pyinstaller yourprogram.py`,回车,
然后在你的程序目录下创建一个dist文件夹,里面包含你要打包的所有相关的东西.
注意:配置文件不会被打包加进去

#### 一些选项
`python pyinstaller.py [opts] yourprogram.py`
主要选项
-F, -onefile 打包成一个exe文件
-D, -onedir 创建一个目录，包含exe文件，但会依赖很多文件（默认选项）
-c, -console, -nowindowed 使用控制台，无界面（默认）
-w, -windowed, -noconsole 使用窗口，无控制台

#### 缺点
打包了很多东西,比较大

### 使用cx_Freeze打包发布程序

#### 安装cx_Freeze
在控制台窗口输入:`pip install cx_Freeze`,回车,然后就自动安装cx_Freeze了.

#### cx_Freeze使用
与py2exe一样,需要在要打包的程序目录下创建一个setup.py, 当然也可以是其他名字
然后在命令行窗口输入:`python setup.py build`,回车,然后就会在程序目录下
创建一个build文件夹,里面打包了所有需要用到的依赖
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

#### 打包成*.msi格式
在命令行窗口输入:`python setup.py bdist_msi`,回车,就会在build目录下生成一个*.msi格式的软件安装包


### 将Python程序打包成.zip文件并发布 
在部署Python程序的时候。一般是把所有的源代码复制到目标机器上。我发现一个更好的办法是把源代码打包成.zip文件，然后直接运行这个.zip文件。比如：`python besteam.zip`
它的秘密是在.zip文件中包含一个\_\_main\_\_.py，当python运行这个zip时，会自动找到它并运行。\_\_main\_\_.py的内容一般是调用主脚本。一行即可，比如:`import besteam`,
如果不想让源代码发布出去，这更是一个好办法。不需要手动地找出编译后的python字节码文件。python提供了一个zipfile. PyZipFile的类自动地将源代码编译成字节码并打包在一起。下面是一个简单的示例脚本：
```python
# -*- coding:utf-8 -*- 
import zipfile, os 
besteamzip = zipfile.PyZipFile("besteam.zip" ,"w", zipfile.ZIP_DEFLATED) 
for filename in ("__main__.py", "besteam.py"): 
    besteamzip.writepy(filename)
for dirname in os.listdir("."):
    initfile=os.path.join(dirname, "__init__.py")
    if os.path.isdir(dirname) and os.path.exists(initfile):
        besteamzip.writepy(dirname)
besteamzip.close()
```
需要注意的是,Python各个版本的字节码是不兼容的。所以，如果运行环境中有多个版本的Python就不能这么搞了，要么制作多个包，要么发布源代码。

### 参考
* [Python依赖打包发布详细](http://www.cnblogs.com/turtle920/p/5370132.html)
* [Freezing Your Code](http://docs.python-guide.org/en/latest/shipping/freezing/)
* [http://www.tuicool.com/articles/Ivuaaq](http://www.tuicool.com/articles/Ivuaaq)
* [http://cx-freeze.readthedocs.io/en/latest/distutils.html](http://cx-freeze.readthedocs.io/en/latest/distutils.html)