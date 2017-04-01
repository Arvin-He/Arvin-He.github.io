---
title: GitBook使用
date: 2017-03-29 14:48:37
tags: Git
categories: 工具
---
# 1. 安装gitbook
1. 安装nodejs和npm
2. 安装gitbook命令行, npm install -g gitbook-cli
3. 查看gitbook是否安装, gitbook -V, 没有安装的话就会自动安装gitbook
4. 安装phantomjs-1.9.7
方法:
windows下:
网上下载phantomjs-1.9.7-windows版本,解压缩,并设置环境变量.
<!--More-->
Linux下:
1.从github上clone一份代码,[phantomjs的github网址](https://github.com/ariya/phantomjs.git),
2.进入这个clone下的仓库,查看发布的版本信息,git tag
3.检出phantom1.9.7版本,git checkout 1.9.7
4.将这个目录放到C盘根目录下,并设置环境变量
```
$ cd phantomjs
$ git checkout 1.9.7  #注意：这里的1.9.7是phantom的版本号，可以由错误报告的第一行找出 
$ ./build.sh --jobs 4
$ sudo cp bin/phantomjs /bin/
$ sudo npm install gitbook-pdf -g  #重新进行安装
```
**注意:**因为gitbook-pdf依赖这个,直接安装gitbook-pdf会报下面的错.
```
Error: connect ETIMEDOUT
    at exports._errnoException (util.js:746:11)
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1010:19)
npm ERR! Linux 3.2.0-4-686-pae
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "install" "gitbook-pdf" "-g"
npm ERR! node v0.12.7
npm ERR! npm  v2.11.3
npm ERR! code ELIFECYCLE
npm ERR! phantomjs@1.9.7-5 install: `node install.js`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the phantomjs@1.9.7-5 install script 'node install.js'.
npm ERR! This is most likely a problem with the phantomjs package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     node install.js
npm ERR! You can get their info via:
npm ERR!     npm owner ls phantomjs
npm ERR! There is likely additional logging output above.
npm ERR! Please include the following file with any support request:
npm ERR!     /home/wangxq/repository/phantomjs/npm-debug.log
```
5. 安装gitbook生成PDF模块, npm install gitbook-pdf -g
6. 如果开始安装gitbook-pdf失败后,在安装了phantomjs之后需要重新安装npm install gitbook-pdf -g
## 2. 使用gitbook
### 2.1 README.md 与 SUMMARY.md创建和编写
README.md 这个文件相当于一本Gitbook的简介。
SUMMARY.md 这个文件是一本书的目录结构，使用Markdown语法
```
$ mkdir mybook
$ vim SUMMARY.md
```
输入
* [简介](README.md)
* [第一章](chapter1/README.md)
 - [第一节](chapter1/section1.md)
 - [第二节](chapter1/section2.md)
* [第二章](chapter2/README.md)
 - [第一节](chapter2/section1.md)
 - [第二节](chapter2/section2.md)
* [结束](end/README.md)

### 2.2 生成图书目录结构
 ```
 gitbook init
 ```

### 2.3 本地图书预览
```
gitbook serve .
```
然后浏览器中输入 http://localhost:4000 就可以预览生成的以网页形式组织的书籍。
在你的图书项目的目录中多了一个名为_book的文件目录，而这个目录中的文件，即是生成的静态
网站内容。

### 2.4 指定图书生成目录
使用build参数生成到指定目录,与直接预览生成的静态网站文件不一样的是，使用这个命令，
你可以将内容输入到你所想要的目录中去：
```
$ mkdir /tmp/gitbook
$ gitbook build --output=/tmp/gitbook
```
## 3. 生成PDF
**注意:**转pdf时需要calibre的ebook-convert组件支持,不然会保存,转换失败.
解决办法:
下载calibre,有windows和linux版的,安装好后,添加环境变量,再重执行gitbook pdf .,新生成pdf
在你的图书的目录中执行下面命令生成PDF.
```
gitbook pdf .
```
**补充：**生成的pdf可以用calibre转成你想要的设备尺寸.

## 4. 生成epub和mobi与电子书
```
gitbook epub .
gitbook mobi .
```
