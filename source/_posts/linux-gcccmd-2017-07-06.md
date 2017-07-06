---
title: Linux之gcc命令
date: 2017-07-06 09:29:44
tags: Linux
categories: 编程
---
### 在linux 下生成.so共享库
要生成共享库，您需要先使用-fPIC（position independent code 位置无关代码）标志来编译C代码, 
`gcc -c -fPIC hello.c -o hello.o`
这将生成一个目标文件（.o），现在你拿它并创建.so文件：
`gcc hello.o -shared -o libhello.so`
你也可以使用:`gcc -shared -o libhello.so -fPIC hello.c` 一步生成.so文件
我还建议添加 `-Wall`选项获取所有的警告，和`-g`选项获取调试信息，到你的gcc命令。
