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