---
title: MongoDB配置
date: 2017-09-27 19:59:15
tags: MongoDB
categories: 数据库
---
### 安装mongodb
安装就免了吧,安装好mongodb,然后配置环境变量

### 创建MongoDB配置文件
**注意:** 配置文件中的所有路径必须要存在, mongodb不会自动给你创建文件夹,所以像data,log,pid等文件夹按照配置文件指定路径手动创建好,否则会报错.
先创建一个配置文件,名称为mongodb.cfg,配置内容为:
```
#数据文件存放位置
dbpath=F:/mongodb/data/db

#日志文件存放位置
logpath=F:/mongodb/data/log/mongodb.log

#PID的路径
pidfilepath=F:/mongodb/pid/mongodb.pid

#端口号
port=27017

#错误日志采用追加模式，配置这个选项后mongodb的日志会追加到现有的日志文件
logappend=true

#启用日志文件，默认启用
journal=true

#这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
quiet=true

#打开28017网页端口（若不开启注释掉即可）
rest=true
```

### 启动mongodb
在终端执行:`mongod --config F:/mongodb/config/mongodb.cfg`
然后在浏览器输入:127.0.0.1:27017,回车会看到网页有"It looks like you are trying to access MongoDB over HTTP on the native driver port.",则表明启动成功

### 注册mongodb服务
如果每次都按照步骤三那样操作，岂不是很麻烦，按照如下命令来创建并启动MongoDB服务，就可以通过windows服务来管理MongoDB的启动和关闭了.
安装mongodb服务,在终端输入: `mongod --config F:/mongodb/config/mongodb.cfg --install --serviceName MongoDB`
启动mongodb服务,在终端输入: `net start MongoDB`, 回车,然后有提示出来:MongoDB服务正在启动, MongoDB服务已经启动成功

错误: 服务名无效
一般由两个原因造成,第一: 可能是你的服务名称拼写错误,第二: 就是没有用管理员权限运行命令.
查看mongodb日志发现: Error connecting to the Service Control Manager: 拒绝访问, 这说明没有用管理员权限运行命令.
使用管理员权限打开cmd, 然后再输入命令并运行, 然后服务就启动成功了.

### 去除mongodb服务
如果需要去除MongoDB服务，执行如下命令：
`mongod --remove --serviceName MongoDB`

### 如何创建用户管理员
用户管理员是第一个要创建的用户。在没有创建任何用户之前，你可以随意创建用户；但数据库中一旦有了用户，那么未登录的客户端就没有权限做任何操作了，除非使用db.auth(username, password)方法登录。

用户管理员的角色名叫 userAdminAnyDatabase，这个角色只能在 admin 数据库中创建。下面是一个例子：
首先,在终端输入:mongo,回车,进入mongo shell, 然后输入命令.
```
> use admin
switched to db admin
> db.createUser({user:"root",pwd:"root",roles:["userAdminAnyDatabase"]})
Successfully added user: { "user" : "root", "roles" : [ "userAdminAnyDatabase" ] }
```

这个例子创建了一个名为 root 的用户管理员。创建完了这个用户之后，我们应该马上以该用户的身份登录：
```
> db.auth("root","root123")
1
db.auth() 方法返回 1 表示登录成功。接下来我们为指定的数据库创建访问所需的账号。
```

### 如何创建数据库用户
首先保证你已经以用户管理员的身份登录 admin 数据库。然后用 use 命令切换到目标数据库，
同样用 db.createUser() 命令来创建用户，其中角色名为 “readWrite”。

普通的数据库用户角色有两种，read 和 readWrite。顾名思义，前者只能读取数据不能修改，后者可以读取和修改。
下面是一个例子：
```
> use test
switched to db test
> db.createUser({user:"testuser",pwd:"testpass",roles:["readWrite"]})
Successfully added user: { "user" : "testuser", "roles" : [ "readWrite" ] }
> db.auth("testuser","testpass")
1
```

这样 MongoDB 的数据安全性就得到保障了，没有登录的客户端将无法执行任何命令。
