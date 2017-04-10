---
title: MySQL免安装配置
date: 2017-04-07 15:56:03
tags: MySQL
categories: 数据库
---

### 1. MySQL下载
下载mysql-5.7.17-winx64.zip,32位机器下载mysql-5.7.17-wix32.zip,
不要下载mysql-installer-community-5.7.17.0.msi.
[mysql下载链接](https://dev.mysql.com/downloads/mysql/)

### 2. 解压mysql
```
#解压目录
D:\Program Files\mysql-5.7.17-winx64
```

### 3. 配置mysql
1. 在mysql解压目录下复制my-default.ini,放到mysql的解压目录,并重命名为my.ini.
2. my.ini里修改为
```
[client]
# port=3306
default-character-set = utf8

[mysql]
default-character-set = utf8

[mysqld]
# basedir 为安装文件解压后的目录 ｜ basedir和datadir 可以使用相对路径
basedir = D:/Program Files/mysql-5.7.17-winx64
# datadir 为用来存放数据的目录
datadir = D:/Program Files/mysql-5.7.17-winx64/data
character_set_server = utf8
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
```

### 4. 添加mysql的环境变量
将`D:\Program Files\mysql-5.7.17-winx64\bin`添加到环境变量.

### 5. 将mysql注册为windows系统服务
1. 从控制台中进入到mysql解压目录下的bin目录
2. 输入命令:`mysqld -install`

### 6. 初始化mysql数据目录
**注意:**mysql5.7解压后没有数据目录,即在解压目录中没有data文件夹.这会导致启动mysql服务失败.
```
mysqld --defaults-file="D:\Program Files\mysql-5.7.17-winx64\my.ini" --initialize-insecure
```
1. 它会初始化 data 目录，在执行此命令前请先把data目录下的所有文件先删除，否则会失败.
2. 可以选择用 --initialize-insecure 或者 --initialize 来初始化，--initialize-insecure 初始化root密码为空，如果用 --initialize来初始化，会产生一个随机密码.
3. 执行成功后，在data目录下会生成mysql，perofrmance_schema，sys等目录文件.

### 7. 启动mysql服务
在控制台输入:`net start mysql`

### 8. mysql一些命令
```
#安装服务
mysqld -install

#启动服务
net start mysql

#进入mysql
mysql -u root -p

#移除mysql服务
mysqld -remove

#停止服务
net stop mysql
```

### 9. 修改root帐号密码
刚安装完成时root账号默认密码为空，此时可以将密码修改为指定的密码。如：123456
```
>mysql –u root
 mysql>show databases;  # 注意:分号不能丢
 mysql>use mysql; # 使用sql数据库,这个是初始化data目录自动生成的
 mysql>set password=password('123456'); # 修改密码
 mysql>FLUSH PRIVILEGES;
 mysql>QUIT
```

### 10. 一些其他命令
```
mysql -u root -p #-u表示用户, -p表示需要输入密码
```

### 11. 一些问题
1. MYSQL服务无法启动
MYSQL服务无法启动的原因是:没有初始化mysql数据目录,mysql5.7解压后,在解压目录中没有data文件夹.

解决方法:
在bin目录下输入命令:`mysqld --initialize-insecure --user=mysql`,然后在mysql根目录下会自动生成一个data文件夹.

2. Error 1045: Access denied for user 'root'@'localhost' (using password: YES)
解决方法:
- > net stop mysql  (停用MySQL服务,没启动的可以省略)
- 找到安装路径 MySQL Server 5.7下的my.ini打开 my.ini,找到  [mysqld]  然后在下面加上
这句： skip\_grant_tables （意思好像是 启动MySQL服务的时候跳过权限表认证  ）
- 启动数据库修改密码了   
>   net start mysql   (启动MySQL服务)--->   mysql  回车   (如果成功，将出现MySQL提示符)
- 输入use mysql; （连接权限数据库）。
- 改密码：update user set authentication_string=password("123") where user="root";（别忘了最后加分号)
- 刷新权限（必须步骤）：flush privileges; 
- 退出 quit
- 将第3 步的 my.ini里的 skip\_grant_tables  去掉（启动MySQL服务的时候不能让他跳过权限表认证 ）
- 重启MySQL ，再进入，使用用户名root和刚才设置的新密码123就可以登录了。 

3. MySQL5.7更改密码时出现ERROR 1054 (42S22): Unknown column 'password' in 'field list'.
新安装的MySQL5.7，登录时提示密码错误，安装的时候并没有更改密码，后来通过免密码登录的方式更改密码，输入update mysql.user  set password=password('123') where user='root'时提示ERROR 1054 (42S22): Unknown column 'password' in 'field list'，
原因:原来是mysql数据库下已经没有password这个字段了，password字段改成了authentication_string.
解决方法:update user set authentication_string=password("123") where user="root";（别忘了最后加分号)
