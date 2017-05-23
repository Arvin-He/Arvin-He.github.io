---
title: Python之Pyinstaller打包发布
date: 2017-05-21 17:07:42
tags: Python
categories: 编程
---
### Pyinstaller安装
控制台窗口输入:`pip install pyinstaller`,回车,然后开始自动安装pyinstaller

### Pyinstaller使用
目录切换到你要打包程序的目录下, 在控制台窗口输入:`pyinstaller yourprogram.py`,回车,
然后在你的程序目录下创建一个dist文件夹,里面包含你要打包的所有相关的东西.
注意:配置文件不会被打包加进去

### 一些选项
`python pyinstaller.py [opts] yourprogram.py`
主要选项
-F, -onefile 打包成一个exe文件
-D, -onedir 创建一个目录，包含exe文件，但会依赖很多文件（默认选项）
-c, -console, -nowindowed 使用控制台，无界面（默认）
-w, -windowed, -noconsole 使用窗口，无控制台

### 缺点
打包了很多东西,比较大
