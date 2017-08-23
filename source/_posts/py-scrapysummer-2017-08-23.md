---
title: Python爬虫总结
date: 2017-08-23 11:38:57
tags: [Python,爬虫]
categories: 编程
---

### 爬虫需要用到的工具
正则表达式
XPATH
beautifulsoup
requests
urllib
urllib2
scrapy

分布式爬虫
学会怎样维护一个所有集群机器能够有效分享的分布式队列就好。最简单的实现是python-rq: https://github.com/nvie/rq

Bloom Filter: Bloom Filters by Example
Bloom Filter. 简单讲它仍然是一种hash的方法，但是它的特点是，它可以使用固定的内存（不随url的数量而增长）以O(1)的效率判定url是否已经在set中。可惜天下没有白吃的午餐，它的唯一问题在于，如果这个url不在set中，BF可以100%确定这个url没有看过。但是如果这个url在set中，它会告诉你：这个url应该已经出现过，不过我有2%的不确定性。注意这里的不确定性在你分配的内存足够大的时候，可以变得很小很少。一个简单的教程:[Bloom Filters by Example](https://llimllib.github.io/bloomfilter-tutorial/)



rq和Scrapy的结合：
后续处理，网页析取，存储(Mongodb)



你只有一台机器。不管你的带宽有多大，只要你的机器下载网页的速度是瓶颈的话，那么你只有加快这个速度。用一台机子不够的话——用很多台吧！当然，我们假设每台机子都已经进了最大的效率——使用多线程（python的话，多进程吧）。

我们把这100台中的99台运算能力较小的机器叫作slave，另外一台较大的机器叫作master，那么回顾上面代码中的url_queue，如果我们能把这个queue放到这台master机器上，所有的slave都可以通过网络跟master联通，每当一个slave完成下载一个网页，就向master请求一个新的网页来抓取。而每次slave新抓到一个网页，就把这个网页上所有的链接送到master的queue里去。同样，bloom filter也放到master上，但是现在master只发送确定没有被访问过的url给slave。Bloom Filter放到master的内存里，而被访问过的url放到运行在master上的Redis里，这样保证所有操作都是O(1)。（至少平摊是O(1)，Redis的访问效率见:LINSERT – Redis)

考虑如何用python实现：
在各台slave上装好scrapy，那么各台机子就变成了一台有抓取能力的slave，在master上装好Redis和rq用作分布式队列。

chrome浏览器 F12开发者工具

selenium
phantomjs


PIL
opencv
pybrain
pyspider

代理IP池


不要用1个IP狂抓
勤换UA
爬取间隔自适应


scrapy/pyspider框架部署


* [https://www.zhihu.com/question/20899988](https://www.zhihu.com/question/20899988)