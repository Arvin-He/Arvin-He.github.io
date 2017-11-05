---
title: Mongodb笔记(三)
date: 2017-09-18 21:05:16
tags: MongoDB
categories: 数据库
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



### 查询和删除某一字段不是数字的记录
```
# 查询某一字段不是数字的记录
db.getCollection('phone').find({"tel": {"$regex": '^[^0-9]+$'}})
# 查询某一字段不是数字的记录的数目
db.getCollection('phone').find({"tel": {"$regex": '^[^0-9]+$'}}).count()
# 删除某一字段不是数字的记录
db.getCollection('phone').remove({"tel": {"$regex": '^[^0-9]+$'}})
```

### 数据库合并

```
# -*- coding: utf-8 -*-
import os
import sys
from pymongo import MongoClient
import gevent
from gevent import monkey
from gevent.pool import Pool
import subprocess
from logger import Logging

monkey.patch_all()


logname = os.path.splitext(os.path.basename(__file__))[0]
logger = Logging(logname)
log = logger.logObject

# 连接mongodb数据库,并返回数据库对象


def connectDB(user, passwd, host, db_name):
    client = MongoClient(
        'mongodb://{}:{}@{}/{}'.format(user, passwd, host, db_name))
    return client[db_name]


db1 = connectDB("xxx", "xxxxxx", "ip:port", "xxxx")
db2 = connectDB("xxxxx", "******",
                "ip:port", "qqqq")
movie1 = db1["xxx1"]
movie2 = db2["xxx2"]


def get_item():
    for index, item in enumerate(movie2.find({}, no_cursor_timeout=True)):
        log.info("获取第{}条数据.".format(index + 1))
        item["time"] = float("{:.3f}".format(int(item["time"] * 1000)))
        yield item


def mergedb(item):
    if movie1.find({"tel": item["tel"], "time": item["time"]}).count() < 1:
        movie1.insert(item)
    else:
        log.info("该数据已经存在!")


def backupdb():
    log.info("开始导出testdb数据库phone集合...")
    dst = os.path.join("data", "{}.dat".format("phone"))
    try:
        subprocess.check_call(["mongoexport",
                               "-h", "ip:port",
                               "-u", "xxx.xx",
                               "-p", "xxxx",
                               "-d", "xxxxx",
                               "-c", "xxxx",
                               "-o", dst])
        log.info("导出{}数据成功".format(dst))
    except Exception as e:
        log.error("导出MongoDB数据出错:{}".format(e))
        return


if __name__ == "__main__":
    pool = Pool(100)
    try:
        pool.map(mergedb, get_item())
    except Exception as e:
        log.error(e)
        sys.exit()

```
