---
title: Linux一些常用命令
date: 2017-09-19 21:46:02
tags: Linux
categories: 编程
---

## scp命令
scp是secure copy的简写，用于在Linux下进行远程拷贝文件或文件夹.scp传输是加密的。可能会稍微影响一下速度。当你服务器硬盘变为只读 read only system时，用scp可以帮你把文件移出来。另外，scp还非常不占资源，不会提高多少系统负荷，在这一点上，rsync就远远不及它了。虽然 rsync比scp会快一点，但当小文件众多的情况下，rsync会导致硬盘I/O非常高，而scp基本不影响系统正常使用。

命令格式：
scp [参数] [原路径] [目标路径]

### 从本地服务器复制到远程服务器：
命令格式：  
```
scp local_file remote_username@remote_ip:remote_folder  
或者  
scp local_file remote_username@remote_ip:remote_file  
或者  
scp local_file remote_ip:remote_folder  
或者  
scp local_file remote_ip:remote_file 
```

第1,2个指定了用户名，命令执行后需要输入用户密码，
第1个仅指定了远程的目录，文件名字不变，
第2个指定了文件名  
第3,4个没有指定用户名，命令执行后需要输入用户名和密码，
第3个仅指定了远程的目录，文件名字不变，
第4个指定了文件名 

### 从本地拷贝文件夹到远程服务器：  
命令格式：  
`scp -r local_folder remote_username@remote_ip:remote_folder`
或者  
`scp -r local_folder remote_ip:remote_folder`
第1个指定了用户名，命令执行后需要输入用户密码；  
第2个没有指定用户名，命令执行后需要输入用户名和密码；

### 从远程服务器复制到本地服务器： 
从远程复制到本地的scp命令与上面的命令相似，只要将从本地复制到远程的命令后面2个参数互换顺序就行了。

`scp arvin@192.168.120.204:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/`

### screen命令
 当使用SSH 或者 telent 远程登录到 Linux 服务器,经常有一些长时间运行的任务，比如系统备份、ftp 传输等等。通常情况下我们都是为每一个这样的任务开一个远程终端窗口，但是他们执行的时间太长了。必须等待它执行完毕，在此期间可不能关掉终端窗口或者断开连接，也不能关机, 否则这个任务就会被杀掉，一切半途而废了。显然,这不是我们所希望的.

screen命令可以远程运行服务器程序并观察程序执行,即使关闭终端或者关闭电脑也不要紧,服务器上的程序也一直在运行.
简单来说，Screen是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。Screen中有会话的概念，用户可以在一个screen会话中创建多个screen窗口，在每一个screen窗口中就像操作一个真实的telnet/SSH连接窗口那样。
```shell
# 创建一个screen会话窗口
screen -S test
# 执行程序
python3 test.py
# 查看正在运行的程序,会显示程序执行的pid
screen -ls
# 查看某个程序在终端的输出, 6245是执行成的pid
screen -r -D 6245
```
**注意:** 如果程序在运行时,不要按下CTRL+C, 这样会中止程序的运行,直接关闭终端窗口就可以了,这样不会关闭程序的运行.

```
其他常用的命令选项有：
-c file	使用配置文件file，而不使用默认的$HOME/.screenrc
-d|-D [pid.tty.host]	不开启新的screen会话，而是断开其他正在运行的screen会话
-h num	指定历史回滚缓冲区大小为num行
-list|-ls	列出现有screen会话，格式为pid.tty.host
-d -m	启动一个开始就处于断开模式的会话
-r sessionowner/ [pid.tty.host]	重新连接一个断开的会话。多用户模式下连接到其他用户screen会话需要指定sessionowner，需要setuid-root权限
-S sessionname	创建screen会话时为会话指定一个名字
-v	显示screen版本信息
-wipe [match]	同-list，但删掉那些无法连接的会话
```
下例显示当前有两个处于detached状态的screen会话，你可以使用`screen -r <screen_pid>`重新连接上.
如果由于某种原因其中一个会话死掉了（例如人为杀掉该会话），这时screen -list会显示该会话为dead状态。使用screen -wipe命令清除该会话.

### kill
kill[参数][进程号]

init进程是不可杀的,
init是Linux系统操作中不可缺少的程序之一。所谓的init进程，它是一个由内核启动的用户级进程。内核自行启动（已经被载入内存，开始运行，并已初始化所有的设备驱动程序和数据结构等）之后，就通过启动一个用户级程序init的方式，完成引导进程。所以,init始终是第一个进程（其进程编号始终为1）。 其它所有进程都是init进程的子孙。init进程是不可杀的！

常用的信号：只有第9种信号(SIGKILL)才可以无条件终止进程，其他信号进程都有权利忽略
```
HUP    1    终端断线
INT     2    中断（同 Ctrl + C）
QUIT    3    退出（同 Ctrl + \）
TERM   15    终止
KILL    9    强制终止
CONT   18    继续（与STOP相反， fg/bg命令）
STOP    19    暂停（同 Ctrl + Z）
```

彻底杀死进程:
命令：`kill –9 3268 `

应注意:
1. 信号使进程强行终止，这常会带来一些副作用，如数据丢失或者终端无法恢复到正常状态。发送信号时必须小心，只有在万不得已时，才用kill信号(9)，因为进程不能首先捕获它。

2. 要撤销所有的后台作业，可以输入kill 0。因为有些在后台运行的命令会启动多个进程，跟踪并找到所有要杀掉的进程的PID是件很麻烦的事。这时，使用kill 0来终止所有由当前shell启动的进程，是个有效的方法。