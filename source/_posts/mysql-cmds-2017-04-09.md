---
title: MySQL基本命令
date: 2017-04-09 16:45:44
tags: MySQL
categories: 数据库
---
### MySQL基本命令
1. 创建一个数据库:`create database gogs;`
2. 显示所有数据库:`show databases;`
3. 使用指定数据库:`use gogs;`
4. 创建表:
```
CREATE TABLE pages (id BIGINT(7) NOT NULL AUTO_INCREMENT, title VARCHAR(200),
content VARCHAR(10000), created TIMESTAMP DEFAULT CURRENT_TIMESTAMP, PRIMARY KEY
(id));
```
**注意:**mysql数据表必须至少有一列,否则不能创建.
每个字段定义由三部分组成：
- 名称（ id 、 title 、 created 等）
- 数据类型（ BIGINT(7) 、 VARCHAR 、 TIMESTAMP ）
- 其他可选属性（ NOT NULL AUTO_INCREMENT ）
在字段定义列表的最后，还要定义一个“主键”（key）.
5. 查看数据表的结构:`describe pages;`
6. 插入数据:
```
insert into pages (title, content) VALUES ("Test page title", "This is some te
st page content. It can be up to 10,000 characters long.");
```
**注意:** pages 表里有四个字段（ id 、 title 、 content 、 created ），但实际上
你只需要插入两个字段（ title 和 content ）的数据即可。因为 id 字段是自动递增的（每
次新插入数据行时 MySQL 自动增加 1），通常不用处理。另外， created 字段的类型是
timestamp ，默认就是数据加入时的时间戳。
7. 查询数据:
```
# 精确查询
select * from pages where id = 2;
# 模糊查询
select * from pages where title like "%test%";
# 返回部分字段查询
select id, title from pages where content like "%page content%";
```
8. 删除数据:`delete from pages where id = 1;`
9. 更新数据:`update pages set title="a new title", content="some new content" where id=2;`


