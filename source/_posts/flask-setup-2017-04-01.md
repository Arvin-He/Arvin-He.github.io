---
title: flaskvenvsetup
date: 2017-04-01 14:46:19
tags: [python,flask]
categories: 编程
---
# 1. flask环境搭建
1. 创建一个文件夹microblog: mkdir microblog
2. 进入microblog目录中:cd microblog
3. 在microblog创建虚拟环境: python -m venv flask, 该命令是在 flask 文件夹中创建一个完整的 Python 环境。
   如果你的python版本低于3.4(包括2.7),需要手动下载安装virtualenv,然后通过命令安装:
   virtualenv flask
4. 虚拟环境是能够激活和关闭的,一个激活的环境可以将它的bin文件夹加到系统路径.但我不喜欢这个特色,所以我从来不激活任何环境,通过输入我想调用的解释器的路径.如想要使用pip工具,则我会输入:flask/scripit/pip install [packagename]
5. 安装flask及扩展
```
$ flask\Scripts\pip install flask
$ flask\Scripts\pip install flask-login
$ flask\Scripts\pip install flask-openid
$ flask\Scripts\pip install flask-mail
$ flask\Scripts\pip install flask-sqlalchemy
$ flask\Scripts\pip install sqlalchemy-migrate
$ flask\Scripts\pip install flask-whooshalchemy
$ flask\Scripts\pip install flask-wtf
$ flask\Scripts\pip install flask-babel
$ flask\Scripts\pip install guess_language
$ flask\Scripts\pip install flipflop
$ flask\Scripts\pip install coverage
```
6. 现在你的microblog文件夹下有一个flask子文件夹,这个flask子文件夹中有python解释器以及flask框架和扩展,环境搭建好后就可以使用flask去创建web应用程序了.


