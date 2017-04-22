---
title: python-reviewcode1-2017-04-22
date: 2017-04-22 09:45:05
tags: [Python]
categories: 编程
---
### 1. 取指定文件所在目录的父目录
```
方式1: os.path.dirname(os.path.dirname(__file__))
方式2: os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
```
dirname是获取输入路径的目录.
尽量用方式2,方式1有风险,当你将目录切到当前的脚本所在的目录时并运行该脚本,输出的目录是空.方式2就不会存在这种问题.看下面的例子
```
print("file1 = {}".format(__file__))
print("file2 = {}".format(os.path.abspath(__file__)))
print("dir1 = {}".format(os.path.dirname(__file__)))
print("dir2 = {}".format(os.path.dirname(os.path.dirname(__file__))))
print("dir3 = {}".format(os.path.dirname(os.path.dirname(os.path.abspath(__file__)))))

输出结果:
F:\2.2>python _build/translate.py
file1 = _build/translate.py
file2 = F:\2.2\_build\translate.py
dir1 = _build
dir2 =
dir3 = F:\2.2
```

### 2. pwd,$pwd与cwd
都指某个进程运行时所在的目录.
pwd 是linux 自带的命令.全称:pathname of the current working directory. 
$PWD 是个系统变量
cwd: 不是系统自带的命令, 但是属于系统的属性.全称: current working directory . 不但在 /proc/{id} 这个目录下可以看到cwd, 在很多其他的编程语言中也可以看到( 例如grunt ).
有时候 pwd 与 $PWD  给出的结果不同.

### 3. decode与encode
