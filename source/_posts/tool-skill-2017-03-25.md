---
title: Windows下工具使用和技巧
date: 2017-03-25 08:52:12
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

## 3. 关于每个文件夹下都有一个文件夹的快捷方式的问题
这是Skypee快捷方式病毒(AutoIt3木马).杀毒软件和安全卫士都弄不干净的.
1. 显示隐藏文件，找到病毒
进入“文件夹选项”界面。点击“查看”，把“隐藏受保护的操作系统文件（推荐）”前面的勾去掉。选择 “显示隐藏的文件、文件夹和驱动器”。在everything中输入"AutoIt3.exe",搜索出所有的AutoIt3.exe并删除
3. 删除开机启动项
打开“任务管理器”，点击“启动”，禁用AutoIt3.exe之类的，病毒还可能伪装成GoogleUpda、WindowsUpdate神马的，右键“属性”可以看到启动项实际位置，把C:\Google下的都删了。
4. 清理注册表
Win+R，再输入regedit，打开注册表编辑器。搜索“C:\Google”，把相关结果删除即可。
注意：在注册器编辑表中，【数据】的选项框如果存在AutoIt之类的字样也一定要将词条注册表数据删除
5. 删除病毒及其创建的快捷方式
我们这里用批处理的方式快速删除。将以下内容保存为Skypee（AutoIt3木马）专杀工具.bat，复制到各个盘中，双击运行即可。
先打开 记事本,复制以下的内容到记事本中。
```
@echo off
title 快捷方式病毒专杀
echo ------ 查找中 ...
attrib -a -s -h -r skypee
del /f /s /q skypee
echo -----Skypee 已删除本盘的毒源------
echo.
ping  -t -n 1 127.0.0.1>nul
echo -----清除此盘一级目录下的病毒快捷方式------
for /f "tokens=*" %%i in ('dir /ad /b *') do (
del /f /s /q  "%%i\%%i.lnk"
)
echo -----清除结束 ^-^------
ping  -t -n 3 127.0.0.1>nul
exit
```
另存为到 你所中招的盘中，文件名就写上：Skypee（AutoIt3木马）专杀工具.bat，文件类型选择：所有
保存文件后，复制到所需盘双击运行Skypee（AutoIt3木马）专杀工具.bat即可删除该盘中的Skypee（AutoIt3木马）的源头和它所生成的快捷方式的了。
 你哪个分区有，就把这个 Skypee（AutoIt3木马）专杀工具.bat 复制到那个盘中运行就可以了。

