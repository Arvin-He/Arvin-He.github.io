---
title: Python删除windows下的目录
date: 2017-08-07 09:40:36
tags: python
categories: python
---
windows启动目录:`C:\Users\aron\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`
在windows下启动目录下添加一个`killkingsoft.py`脚本,然后每次开机自动执行该脚本.
注意:在windows下启动目录下的脚本或者bat文件都会在开机时自动执行.


之前使用`shutil.rmtree  os.remove  os.rmdir`都没有成功,都报出如下错误:
`PermissionError: [WinError 5] 拒绝访问。: 'c:\\ProgramData\\kingsoft'`
后来找到解决办法,如下:
```python
# usr/bin/python3
# -*- coding:utf-8 -*-
   
import errno, os, stat, shutil


kingSoft = os.path.join("c:\\", "ProgramData", "kingsoft")


def handleRemoveReadonly(func, path, exc):
  excvalue = exc[1]
  if func in (os.rmdir, os.remove) and excvalue.errno == errno.EACCES:
      os.chmod(path, stat.S_IRWXU| stat.S_IRWXG| stat.S_IRWXO) # 0777
      func(path)
  else:
      raise


if os.path.exists(kingSoft):
    shutil.rmtree(kingSoft, ignore_errors=False, onerror=handleRemoveReadonly)
    print("delete kingsoft success")
else:
    print("no such dir or file")
```