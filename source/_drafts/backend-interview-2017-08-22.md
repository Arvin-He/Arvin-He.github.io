---
title: 后端面试
tags:
---


一.语言 
1.推荐一本看过最好的python书籍？ 拉开话题好扯淡 
2.谈谈python的装饰器，迭代器，yield？ 
3.标准库线程安全的队列是哪一个？不安全的是哪一个？logging是线程安全的吗？ 
4.python适合的场景有哪些？当遇到计算密集型任务怎么办？ 
5.python高并发解决方案？我希望听到twisted->tornado->gevent，能扯到golang,erlang更好

二.操作系统 
可以直接认为是linux，毕竟搞后端的多数是和linux打交道。 
1.tcp/udp的区别？tcp粘包是怎么回事，如何处理？udp有粘包吗？ 
2.time_wait是什么情况？出现过多的close_wait可能是什么原因？ 
3.epoll,select的区别？边缘触发，水平触发区别？

三.存储 
存储可能包含rdbms，nosql以及缓存等，我以mysql,redis举例 mysql相关 
1.谈谈mysql字符集和排序规则？ 
2.varchar与char的区别是什么？大小限制？utf8字符集下varchar最多能存多少个字符 
3.primary key和unique的区别？ 
4.外键有什么用，是否该用外键？外键一定需要索引吗？ 
5.myisam与innodb的区别？innodb的两阶段锁定协议是什么情况？ 
6.索引有什么用，大致原理是什么？设计索引有什么注意点？ 

redis相关 
1.什么场景用redis，为什么mysql不适合？ 
2.谈谈redis的事务？用事务模拟原子+1操作？原子操作还有其它解决方案吗？ 
3.redis内存满了会怎么样？

四.安全 
web安全相关 
1.sql注入是怎么产生的，如何防止？ 
2.xss如何预防？htmlescape后能否避免xss? 
3.csrf是什么？django是如何防范的？ 

密码技术 
1.什么是分组加密？加密模式有哪些？ecb和cbc模式有什么区别？为什么需要iv向量？ 
2.简单说说https的过程？ 
3.对称加密与非对称加密区别？ 
3.如何生成共享秘钥？ 如何防范中间人攻击？

五.杂 
是否关注新技术啊？golang，rust是否了解？numpy,pandas是啥鸟？ 
是否紧跟时代潮流？逛不逛微博，刷不刷知乎？


- 谈谈装饰器，迭代器，yield，内存管理等    
- Python高并发解决方案    
- 计算密集型，IO密集型任务怎么办    
- Tcp/Udp协议，Http协议    
- sql，cache, nosql    
- web安全相关，sql注入，xss等


解释一下 WSGI 和 FastCGI 的关系
解释一下 Gevent 和 threading / multiprocessing 的关系
解释一下 cookie 和 session 的关系，以及 xsrf 的攻击和防范方法
解释一下 Django 和 Tornado 的关系、差别
考考数据库知识，SQL 语法和调优，SQLAlchemy 的用法



数据结构
算法
网络编程
socket
http/tcp/ip
并发
异步
数据库(mysql/monogodb/redis)多表查询

python
flask/django/tonado/webpy
celery


java

shell脚本

js/css/html

vue/react

nodejs

设计模式

