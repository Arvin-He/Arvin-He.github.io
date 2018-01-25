---
title: Flask用户登录
date: 2017-04-16 17:19:00
tags: flask
categories: python
---
Flask用户登录是建立 web 表单和数据库的联系，并且编写登录系统。

### 配置
对于登录系统，会使用到两个扩展，Flask-Login 和 Flask-OpenID,
Flask-OpenID 扩展需要一个存储文件的临时文件夹的路径。对此，我们提供了一个 tmp 文件夹的路径。

### 重构用户模型
Flask-Login 扩展需要在我们的 User 类中实现一些特定的方法。但是类如何去实现这些方法却没有什么要求。

### user_loader回调
必须编写一个函数用于从数据库加载用户。这个函数将会被 Flask-Login 使用

### 登录视图函数
oid.loginhandle 告诉 Flask-OpenID 这是登录视图函数。
在函数开始的时候，用检查 g.user 是否被设置成一个认证用户，如果是的话将会被重定向到首页。这里的想法是如果是一个已经登录的用户的话，就不需要二次登录了。
Flask 中的 g 全局变量是一个在请求生命周期中用来存储和共享数据。将登录的用户存储在这里(g)。
在 redirect 调用中使用的 url_for 函数是定义在 Flask 中，以一种干净的方式为一个给定的视图函数获取 URL。 让 Flask 为你构建 URLs是一个很好的选择。
OpenID 认证异步发生。如果认证成功的话，Flask-OpenID 将会调用一个注册了 oid.after_login 装饰器的函数。如果失败的话，用户将会回到登陆页面。

### Flask-OpenID 登录回调
这里是 after_login 函数

### 全局变量g.user
在登录视图函数中我们检查 g.user 为了决定用户是否已经登录.
为了实现这个我们用 Flask 的 before\_request 装饰器。任何使用了 before_request 装饰器的函数在接收请求之前都会运行

### 登出
只需要检查有效的用户是否被设置到 g.user 以及是否我们已经添加了登出链接。