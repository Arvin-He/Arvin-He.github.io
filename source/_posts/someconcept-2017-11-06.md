---
title: Linux IO模式及 select、poll、epoll详解
date: 2017-11-06 20:49:40
tags: [Linux] 
categories: Linux
---

##  epoll, kqueue, select, iocp的概念
epoll


## 阻塞, 非阻塞忙轮询


## 缓冲区,内核缓冲区
缓冲区的引入是为了减少频繁I/O操作而引起频繁的系统调用，当你操作一个流时，更多的是以缓冲区为单位进行操作，这是相对于用户空间而言。对于内核来说，也需要缓冲区。


## 用户态, 内核态
