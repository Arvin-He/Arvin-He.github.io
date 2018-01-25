---
title: Python的sys.path, PYTHONPATH, os.environ的作用
date: 2017-08-13 19:06:37
tags: python
categories: python
---

### Python搜索模块的路径：
1. 程序的主目录
2. PTYHONPATH目录（如果已经进行了设置）
3. 标准连接库目录（一般在/usr/local/lib/python2.X/）
4. 任何的.pth文件的内容（如果存在的话）.新功能，允许用户把有效果的目录添加到模块搜索路径中去, `.pth`后缀的文本文件中一行一行的地列出目录。

这四个组建组合起来就变成了sys.path了

### 关于sys.path
在python 环境下使用sys.path.append(path)添加相关的路径，但在退出python环境后自己添加的路径就会自动消失.
如何将路径“永久"添加到sys.path?
1. 将自己做的py文件放到 site_packages 目录下,但是这样做会导致一个问题，即各类模块都放到此文件夹的话，会导致乱的问题.
2. 使用pth文件，在 site-packages 文件中创建 .pth文件，将模块的路径写进去，一行一个路径，但存在管理上的问题，而且不能在不同的python版本中共享。
3. 使用PYTHONPATH环境变量，在这个环境变量中输入相关的路径，不同的路径之间用逗号（英文的！)分开，如果PYTHONPATH 变量还不存在，可以创建它.路径会自动加入到sys.path中，而且可以在不同的python版本中共享，应该是一样较为方便的方法。

```python
In [16]: sys.path
Out[16]:
['',
 'C:\\Program Files\\Python36\\Scripts\\ipython.exe',
 'c:\\program files\\python36\\python36.zip',
 'c:\\program files\\python36\\DLLs',
 'c:\\program files\\python36\\lib',
 'c:\\program files\\python36',
 'c:\\program files\\python36\\lib\\site-packages',
 'c:\\program files\\python36\\lib\\site-packages\\win32',
 'c:\\program files\\python36\\lib\\site-packages\\win32\\lib',
 'c:\\program files\\python36\\lib\\site-packages\\Pythonwin',
 'c:\\program files\\python36\\lib\\site-packages\\IPython\\extensions',
 'C:\\Users\\Arvin\\.ipython']
```

### 关于PYTHONPATH


### 关于os.environ
```
environ是一个字符串对应环境的映像对象
os.environ.keys() 主目录下所有的key
os.environ 显示key+内容

# windows：
· os.environ['HOMEPATH']:当前用户主目录。
os.environ['TEMP']:临时目录路径。
os.environ[PATHEXT']:可执行文件。
os.environ['SYSTEMROOT']:系统主目录。
os.environ['LOGONSERVER']:机器名。
os.environ['PROMPT']:设置提示符。
# linux：
os.environ['USER']:当前使用用户。
os.environ['LC_COLLATE']:路径扩展的结果排序时的字母顺序。
os.environ['SHELL']:使用shell的类型。
os.environ['LAN']:使用的语言。
os.environ['SSH_AUTH_SOCK']:ssh的执行路径。
```