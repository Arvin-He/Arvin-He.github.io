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