---
title: mongodb笔记(三)
date: 2017-09-18 21:05:16
tags: MongoDB
categories: 编程
---

### mongodb突然无法打开
检查mongodb数据所在的文件夹下是否有一个类似"locked file",删掉这个文件,然后再开启mongodb

### 再说插入操作
单条插入
批量插入

### find操作
查询是用的最多的操作了, 常用的有2类:
1. >, >=, <, <=, !=, =
2. And，OR，In，NotIn

这些操作在mongodb里面都有对应封装.
1. "$gt", "$gte", "$lt", "$lte", "$ne", "" 
这些与上面 >, >=, <, <=, !=, = 这6个符号操作一一对应
2. "", "$or", "$in"，"$nin"
与 And，OR，In，NotIn 操作一一对应

正则表达式匹配
在mongodb中还有一个特殊的匹配，那就是支持正则表达式.

$where操作

### update操作
整体更新
局部更新

局部更新:
mongodb中已经给我们提供了两个修改器： $inc 和 $set。

$inc修改器
$inc也就是increase的缩写，自增$inc指定的值，如果“文档”中没有此key，则会创建key。

$set修改器


### upsert操作
upsert操作就是说：如果我没有查到，我就在数据库里面新增一条，其实这样也有好处，就是避免了我在数据库里面判断是update还是add操作，使用起来很简单将update的第三个参数设为true即可。

### 批量更新
在mongodb中如果匹配多条，默认的情况下只更新第一条，那么如果我们有需求必须批量更新，那么在mongodb中实现也是很简单的，在update的第四个参数中设为true即可.


### 查询某一字段重复的记录


### 查询某一字段重复的记录的数目

