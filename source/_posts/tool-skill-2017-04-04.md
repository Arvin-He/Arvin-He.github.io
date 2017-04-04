---
title: Windows下工具使用和技巧
date: 2017-04-04 08:52:12
tags: [Tools]
categories: 工具
---
## 1. cmder简介
cmder是windows下一款很好用的命令行工具,支持绝大部分linux命令,自带git和vim.

### 1.1 解决cmder中文乱码问题
win+alt+p打开设置面板，找到Startup-Envrioment选项
在下面的文本框里添加一行 set LANG=zh_CN.UTF-8
然后重启cmder

### 1.2 解决文字重叠问题
Win + Alt + P 打开设置界面 
在mian > font > monospce 的勾去掉.

### 1.3 设置cmder分屏显示
设置快捷键
打开设置面板选择keys & Macro,或者右击标题栏,在help中的Hotkeys中做如下设置
找到Split: spliter Duplicate active "shell" to right:split,并选中
在下面设置快捷键,如ctrl+Alt+1
保存设置,然后重启.

## 2. 通过命令行追加环境变量
例如添加C:\phantomjs
```
SETX PATH "%PATH%;C:\\phantomjs" /M
```
如果写成下面的形式,则通过窗口查看机器的环境变量时会发现已经用C:\\phantomjs覆盖以前所有的环境变量,之前的环境变量全部没了.
```
SETX PATH C:\\phantomjs /M
```
**注意:**重启才能生效,只要当前还没有关机或者重启机器,则之前的环境变量还是能够在命令行中显示,
覆盖的环境变量还没有生效,我们将之前的环境变量导入到文件中再拷贝回去就好了,然后重启机器生效.

