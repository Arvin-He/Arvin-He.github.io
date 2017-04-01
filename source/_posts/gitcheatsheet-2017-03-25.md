---
title: Git命令使用
date: 2017-03-25 09:47:37
tags: Git
categories: 工具
---
[Git Pro 第二版](https://git-scm.com/book/zh/v2)
# 1. 初次运行 Git 前的配置
```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```
<!--More-->
**注意:**
如果用了 --global 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。
如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 --global 选项重新配置即可，新的设定保存在当前项目的 .git/config 文件里。

# 2. 查看git配置信息
```
git config --list #检查已有的配置信息
git config user.name #查看git用户名
git config user.email #查看git邮箱
```
# 3. 在本地保存github帐号的用户名和密码,不用每次输入用户名和密码
使用https协议时,在向远程仓库push提交内容时,总是每次要求输入你的github用户名和密码
解决办法:
1. 设置记住密码（默认15分钟）
```
git config --global credential.helper cache
```
2. 自己设置记住密码时间,比如一小时
```
git config credential.helper ‘cache --timeout=3600’
```
3. 长期存储密码
```
git config --global credential.helper store
```
配置好后在 .gitconfig 文件中可以看到.

# 4. 配置github上ssh密匙,不用每次输入用户名和密码
使用ssh协议时,也要输入用户名和密码,要想不用每次输入用户名和密码,则在你的本地机器生成ssh的私匙和公匙,
私匙保存在本地机器,公匙保存在你的github网站上的帐号设置上.
1. 创建SSH Key
在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，
如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
```
$ ssh-keygen -t rsa -C "youremail@example.com"
```
2. 登陆GitHub，打开“Settings”，“SSH and GPG Keys”页面
3. 点击“New SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容

# 5. 不小心纳入仓库后，要移除跟踪但不删除文件，以便稍后在 .gitignore 文件中补上，用 --cached 选项即可
```
git rm --cached readme.txt
```

# 6. 修改.gitignore并更新本地和远程仓库
1. 在.gitignore修改过滤规则,并保存.
2. 更新本地和远程仓库
```
# 注意有个点“.”
git rm -r --cached .
git add -A
git commit -m "update .gitignore"
```
# 7. 丢弃本地修改,使用远程仓库的内容
```
git fetch --all
git reset --hard origin/master
git fetch 只是下载远程的库的内容，不做任何的合并,
git reset 把HEAD指向刚刚下载的最新的版本
```

