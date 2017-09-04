---
title: Python程序打包问题
date: 2017-09-01 14:33:39
tags: Python
categories: 编程
---
### ImportError: DLL load failed: The specified module could not be found.
win7-64bit或win10-64bit打包python32位程序在win7-32bit系统上运行报错:ImportError: DLL load failed: The specified module could not be found.

Finally,I find the solution:
Install [Microsoft Visual C++ 2015 Redistributable Update 3 x86](https://www.microsoft.com/de-at/download/details.aspx?id=48145).

注意:
1. 选择X86版本
2. vc_redist.x86.exe 一定要选择update 3版本,之前的版本还是会报错.


### 参考
* [stackoverflow](https://github.com/tensorflow/tensorflow/issues/7995)

