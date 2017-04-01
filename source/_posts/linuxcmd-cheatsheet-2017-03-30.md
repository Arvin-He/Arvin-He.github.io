---
title: linux命令使用
date: 2017-03-30 21:10:18
tags: Linux
---
# 1. 查看文件/文件夹占用空间大小和设备的空间使用率
1. du(disk usage),顾名思义,查看目录/文件占用空间大小
<!--More-->
```
#查看当前目录下的所有目录以及子目录的大小
$ du -h
$ du -ah
#-h:用KB、MB、GB的人性化形式显示文件容量大小
#-a:显示目录和文件

du -h tmp
du -ah tmp
#只查看当前目录下的tmp目录(包含子目录)的大小

#查看当前目录及其指定深度目录的大小
du -h –-max-depth=0
#-–max-depth＝n:只深入到第n层目录，此处设置为0，即表示不深入到子目录
```
du命令的一些常用参数:
```
-a或-all 显示目录中个别文件的大小
-b或-bytes 显示目录或文件大小时，以byte为单位
-c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
-D或--dereference-args 显示指定符号连接的源文件大小
-h或--human-readable 以K，M，G为单位，提高信息的可读性
-k或--kilobytes 以1024 bytes为单位
-l或--count-links 重复计算硬件连接的文件
-L或--dereference 显示选项中所指定符号连接的源文件大小
-m或--megabytes 以1MB为单位
-s或--summarize 仅显示总计
-S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小
-X<文件>或--exclude-from=<文件>
--exclude=<目录或文件> 略过指定的目录或文件
--max-depth=<目录层数> 超过指定层数的目录后，予以忽略
```
2. df 用于查看设备的空间使用率
查看设备使用率
```
$ df -lh    
```