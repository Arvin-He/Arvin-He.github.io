---
title: Python之程序打包发布
date: 2017-05-08 11:20:24
tags: Python
categories: 编程
---

### Windows上python程序打包发布
Python是一个脚本语言，被解释器解释执行。它的发布方式：
* .py文件：对于开源项目或者源码没那么重要的，直接提供源码，需要使用者自行安装Python并且安装依赖的各种库。（Python官方的各种安装包就是这样做的）
* .pyc文件：有些公司或个人因为机密或者各种原因，不愿意源码被运行者看到，可以使用pyc文件发布，pyc文件是Python解释器可以识别的二进制码，故发布后也是跨平台的，需要使用者安装相应版本的Python和依赖库。
* 可执行文件：对于非码农用户或者小白，让他装个Python同时还要折腾一堆依赖库，那简直是个灾难。对于此类用户，最简单的方式就是提供一个可执行文件，只需要把用法告诉Ta即可。比较麻烦的是需要针对不同平台需要打包不同的可执行文件（Windows,Linux,Mac,...）。

各种打包工具的对比如下：

|Solution	|Windows	|Linux	|OS X	|Python 3	|License	|One-file mode	|Zipfile import	|Eggs	|pkg_resources support|
|:--------|---------|-------|-----|---------|---------|---------------|---------------|-----|---------------------|
|bbFreeze	  |yes	|yes	|yes	|no	  |MIT	|no	  |yes	|yes	|yes|
|py2exe	    |yes	|no	  |no	  |yes	|MIT	|yes	|yes	|no	  |no|
|pyInstaller|yes	|yes	|yes	|yes	|GPL	|yes	|no	  |yes	|no|
|cx_Freeze	|yes	|yes	|yes	|yes	|PSF	|no	  |yes	|yes	|no|
|py2app	    |no	  |no	  |yes	|yes	|MIT	|no	  |yes	|yes	|yes|

PS.其中pyInstaller和cx\_Freeze都是不错的，stackoverflow上也有人建议用cx\_Freeze，说是更便捷些。pkg\_resources新版的pyInstaller貌似是支持的。

### py2exe打包发布
1. 安装py2exe: `pip install py2exe`
2. 准备好你要打包的程序,如hello.py
```python
# hello.py
print("hello")
```
3. 创建安装脚本程序（setup.py）
py2exe工具只是在原来Distutils工具之上进行扩展，并且进行一步优化，如果使用过Distutils工具，就会知道下面的命令行：`python setup.py install`

所以在py2exe工具里也需要一个像setup.py的脚本，脚本具体内容如下：
```python
#python 3.4
from distutils.core import setup
import py2exe

setup(console=['hello.py'])
```

在这个脚本里，第二行代码是从distutils库里导入setup函数。第三行代码是导出入py2exe模块。第四行代码是空行，用来分隔导入模块与实际运行代码。第五行代码是调用setup函数，主要创建控制台应用程序，它的入口主文件是hello.py文件。

3. 运行脚本（setup.py）文件
在cmd.exe的窗口里运行下面的命令：`F:\temp\py>python setup.py py2exe`
```
running py2exe
  3 missing Modules
  ------------------
? readline                            imported from cmd, code, pdb
? win32api                            imported from platform
? win32con                            imported from platform
Building 'dist\hello.exe'.
Building shared code archive 'dist\library.zip'.
Copy c:\windows\system32\python34.dll to dist
Copy C:\Python34\DLLs\unicodedata.pyd to dist\unicodedata.pyd
Copy C:\Python34\DLLs\_socket.pyd to dist\_socket.pyd
Copy C:\Python34\DLLs\_ctypes.pyd to dist\_ctypes.pyd
Copy C:\Python34\DLLs\_bz2.pyd to dist\_bz2.pyd
Copy C:\Python34\DLLs\pyexpat.pyd to dist\pyexpat.pyd
Copy C:\Python34\DLLs\_ssl.pyd to dist\_ssl.pyd
Copy C:\Python34\DLLs\_hashlib.pyd to dist\_hashlib.pyd
Copy C:\Python34\DLLs\select.pyd to dist\select.pyd
Copy C:\Python34\DLLs\_lzma.pyd to dist\_lzma.pyd
```
运行这个命令成功之后，会在当前的目录下面创建一个发布的目录dist，所有需要发布的文件就会拷贝到此目录下面。
4. 执行生成的exe程序
经过上面的步骤，就可以进入目录dist下面进行运行exe程序了,
```
F:\temp\py\dist>hello.exe

hello World
```
运行成功之后，与前面使用python hello.py是一样的结果，不过这个目录内容就可以发布到不同的电脑上进行运行，并不再需要安装python的安装程序。

### cx_Freeze打包发布程序

### pyInstaller打包发布程序

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


### Linux上python程序打包发布



### 参考
* [Python依赖打包发布详细](http://www.cnblogs.com/turtle920/p/5370132.html)
* [Freezing Your Code](http://docs.python-guide.org/en/latest/shipping/freezing/)
* [http://www.tuicool.com/articles/Ivuaaq](http://www.tuicool.com/articles/Ivuaaq)