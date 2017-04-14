---
title: Python使用
date: 2017-04-01 15:41:13
tags: Python
categories: 编程
---
### 1. windows下Python2与Python3共存的使用方法
windows下同时安装了Python2.7.13与Python3.6后,默认的使用python2.
1. 命令行分别使用python2和python3
- 调用python2,使用 py -2
- 调用python3,使用 py -3

2. 使用pip安装库
- 安装到Python2时，就使用 pip2 install [name]
- 安装到Python3时，就使用pip3 install [name]

### 2. pip工具使用
pip 命令使用:
  pip <command> [options]
```
Commands:
  install                     Install packages.
  download                    Download packages.
  uninstall                   Uninstall packages.
  freeze                      Output installed packages in requirements format.
  list                        List installed packages.
  show                        Show information about installed packages.
  check                       Verify installed packages have compatible dependencies.
  search                      Search PyPI for packages.
  wheel                       Build wheels from your requirements.
  hash                        Compute hashes of package archives.
  completion                  A helper command used for command completion.
  help                        Show help for commands.
```

### 3. Windows下python非官方第三方包地址
[Windows下python非官方第三方包地址](http://www.lfd.uci.edu/~gohlke/pythonlibs/#scipy)

### 4. python中的对象引用
python中set,list,tuple,dict都是表示对象,这些对象在给新对象赋值的时候是传递引用,并不是传值.

### 5. 返回值问题
有时在一个循环里计算,然后将计算的结果一起返回,注意return语句的位置,不能放在循环里.尤其注意在循环语句比较多的情况下.

### 6. 列表过滤
当你想要过滤掉列表中的元素,优先选用列表表达式.不要用循环加判断的方式.

### 7. 闭包中的自由变量
