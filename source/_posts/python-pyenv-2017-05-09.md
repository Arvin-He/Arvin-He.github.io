---
title: Python之pyenv与virtualenv使用
date: 2017-05-09 09:45:02
tags: python
categories: python
---
### pyenv 与 virtualenv
多个python版本共存有2种方式:
python版本切换的工具—pyenv, pyenv目前还不支持windows,只支持Linux.
另外一个工具virtualenv则提供了一种功能,就是将一个目录建立为一个虚拟的python环境,
用户可以建立多个虚拟环境,每个环境里面的python版本可以是不同的,也可以是相同的,而且环境之间相互独立。

### pyenv 与 virtualenv区别
pyenv 是针对 python 版本的管理，通过修改环境变量的方式实现；
virtualenv 是针对python的包的多版本管理，通过将python包安装到一个模块来作为python的包虚拟环境，通过切换目录来实现不同包环境间的切换。

### pyenv安装与使用
自动安装: pyenv 提供了自动安装的工具，执行命令安装即可：
```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```
手动安装:将 pyenv 检出到你想安装的目录。建议路径为：$HOME/.pyenv
```
$ cd
$ git clone git://github.com/yyuu/pyenv.git .pyenv
```
添加环境变量。PYENV_ROOT 指向 pyenv 检出的根目录，并向 $PATH 添加 $PYENV_ROOT/bin 以提供访问 pyenv 这条命令的路径
```
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
# 向 shell 添加 pyenv init 以启用 shims 和命令补完功能
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```
这里的 shell 配置文件（~/.bash_profile）依不同 Linux作修改——Zsh：~/.zshenv；Ubuntu：~/.bashrc 
重启 shell（因为修改了 $PATH）,`$ exec $SHELL`

### pyenv常用命令
```
$ pyenv install --list # 该命令会列出可以用 pyenv 安装的 Python 版本。列表很长，仅列举其中几个:
$ pyenv versions # 查看系统当前安装的python列表
$ pyenv install -v 3.5.1 # 安装python
$ pyenv uninstall 2.7.3 # 卸载python
$ pyenv update          # 更新 pyenv 及其插件
$ pyenv rehash # 创建垫片路径
为所有已安装的可执行文件 （如：~/.pyenv/versions//bin/） 创建 shims，
因此，每当你增删了 Python 版本或带有可执行文件的包（如 pip）以后，都应该执行一次本命令）
```

### python切换
```
$ pyenv global 3.4.0 – 设置全局的 Python 版本，通过将版本号写入 ~/.pyenv/version 文件的方式。
$ pyenv local 2.7.3 – 设置面向程序的本地版本，通过将版本号写入当前目录下的 .python-version 文件的方式。通过这种方式设置的 Python 版本优先级较 global 高。
pyenv 会从当前目录开始向上逐级查找 .python-version 文件，直到根目录为止。若找不到，就用 global 版本。
$ pyenv shell pypy-2.2.1 – 设置面向 shell 的 Python 版本，通过设置当前 shell 的 PYENV_VERSION 环境变量的方式。这个版本的优先级比 local 和 global 都要高。–unset 参数可以用于取消当前 shell 设定的版本。
$ pyenv shell --unset
```
python优先级
shell > local > global

### virtualenv安装与使用
**注意:**创建的虚拟环境是和你当前安装的python版本是一致的,它不能用作不同python版本的切换, 如果你要在windows下安装不同的python版本,就只能再安装一个python版本.
virtualenv主要用来管理包不同版本来设定独立的python环境,与不同版本包做区分.
在命令行窗口输入:`pip install virtualenv`,回车

### 创建虚拟环境
如果你已经安装好virtualenv, 则进入到工程目录中,接着使用如下的命令创建一个虚拟环境
```
$ python -m venv flask
```
如果你的python版本低于3.4,则需要安装 virtualenv.py,
```
# windows
pip install virtualenv
virtualenv flask
# linux
sudo apt-get install python-virtualenv
virtualenv flask
```
这样就创建了一个完整的 Python 环境.
虚拟环境是能够激活以及停用的，如果需要的话，一个激活的环境可以把它的 bin 文件夹加入到系统路径。
我个人是不喜欢这种特色，所以我从来不激活任何环境,相反我会直接输入我想要调用的解释器的路径。如下:
调用虚拟环境的pip安装python的包
```
# windows
flask\Scripts\pip install flask
# linux
flask/bin/pip install flask
```


### 参考
* [http://www.pylixm.cc/posts/2016-06-19-Virtualenv-install.html](http://www.pylixm.cc/posts/2016-06-19-Virtualenv-install.html)
