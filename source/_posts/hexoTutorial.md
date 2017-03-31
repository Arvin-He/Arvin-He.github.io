---
title: hexo搭建github博客
date: 2017-03-25 15:26:36
tags: [hexo,Git]
categories: 工具
---
# 1. 搭建hexo
## 1.1 准备条件
1. 安装git
2. 有github帐号,并有仓库username.github.io, 如Arvin-He.github.io
3. 本地机器保存github的帐号和密码

## 1.2 安装hexo
1. 安装nodejs
2. 安装hexo, npm install -g hexo, 或者 npm install hexo-cli -g
<!--More-->
## 1.3 hexo配置文件和next主题配置
github page默认主题是landscape,这个主题不怎么喜欢,于是更换next主题

### 1.3.1 下载next主题
```
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 1.3.2 更改next主题配置
**注意:** 在 Hexo中的next主题中有两份主要的配置文件，名称都是 _config.yml。 
其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；
另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。
如何配置请参考官方文档:[hexo-next官方配置教程](http://theme-next.iissnan.com/getting-started.html)


## 1.4 使用hexo
新建一个目录如:F:/blog/,或者直接将git clone Arvin-He.github.io仓库,并创建一个分支,比如source
cd Arvin-He.github.io
hexo init       //文件夹自动生成建网站所需的文件
npm install     //在文件夹下安装node_modules,即安装依赖
hexo g          //生成静态文件
hexo s          //本地运行
hexo d          //部署到github上去

---
# 2. hexo命令详解
## 2.1 hexo命令
hexo init 
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo clean 清除生成的静态文件
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口(默认端口4000，'ctrl + c'关闭server)
hexo deploy #将.deploy目录部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
hexo deploy -g  #生成加部署
hexo server -g  #生成加预览<br>
常用命令的简写
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy<br>
常用组合
hexo d -g #生成部署
hexo s -g #生成预览

---
# 3. 搭建hexo过程中遇到的问题

## 3.1 hexo无法上传到github, 但在本地localhost:4000是可以打开的
On branch master
nothing to commit, working directory clean
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error<br>

原因:
本地没有保存github用户名和密码,因为hexo d,是直接将生成的静态文件部署到github上去,一步到位的,意思就是部署的时候就要能拿到github的用户名和密码,如果拿不到就报错,部署不上去.
**注意:** 使用的是https协议,因为当没有在本地保存你的github帐号的用户名
和密码时,使用https协议push内容到github上去时就要每次手动输入github帐号上用户名和密码.
而hexo d部署是一步到位的,中间没有机会让你输入你的github账户用户名和密码.

解决办法:
1. 使用https协议时,本地保存你的github账户的用户名和密码
deploy:
  type: git
  repository: https://github.com/Arvin-He/Arvin-He.github.io.git
  branch: master<br>
2. 使用ssh协议,本地生成ssh key,并将公匙放到你的github上
deploy:
  type: git
  repository: ssh://git@github.com/Arvin-He/Arvin-He.github.io.git
  branch: master

## 3.2 hexo部署失败 ERROR Deployer not found: git
原因: 缺少nodejs的依赖
解决办法: 在 blog 目录里 执行 npm install hexo-deployer-git --save 

## 3.3 Error: fatal: Not a git repository (or any of the parent directories): .git
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: fatal: Not a git repository (or any of the parent directories): .git
    at ChildProcess.<anonymous> (F:\blogs\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:194:7)
    at ChildProcess.cp.emit (F:\blogs\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:899:16)
    at Socket.<anonymous> (internal/child_process.js:342:11)
    at emitOne (events.js:96:13)
    at Socket.emit (events.js:191:7)
    at Pipe._handle.close [as _onclose] (net.js:513:12)
FATAL fatal: Not a git repository (or any of the parent directories): .git
Error: fatal: Not a git repository (or any of the parent directories): .git
    at ChildProcess.<anonymous> (F:\blogs\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:194:7)
    at ChildProcess.cp.emit (F:\blogs\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:899:16)
    at Socket.<anonymous> (internal/child_process.js:342:11)
    at emitOne (events.js:96:13)
    at Socket.emit (events.js:191:7)
    at Pipe._handle.close [as _onclose] (net.js:513:12)
**解决办法:**
    删除.deploy_git/ 文件夹,然后再次,hexo deploy, 就好了.

## 3.4 ERROR Local hexo not found in F:\Arvin-He.github.io
Hexo搭建博客之后用Git已经将所有的source都同步到了git上，在另一台电脑上将源代码clone下来之后，直接执行hexo g,出现错误.
F:\Arvin-He.github.io (source) (hexo-site@0.0.0)
$ hexo g<
ERROR Local hexo not found in F:\Arvin-He.github.io
ERROR Try running: 'npm install hexo --save'
**原因:** 在另外一台机器上clone下来的内容,缺少hexo的依赖,因为在.gitignore里忽略将node_nodules文件夹push到github上去.就是hexo所依赖的nodejs的模块,所以用npm重新安装即可。<br>
**解决办法:**
cd F:\Arvin-He.github.io
npm install 
hexo g

## 3.5 执行hexo g或hexo本地测试运行重启后页面空白,提示 : WARN No layout: index.html
此时页面都是白的，使用hexo clean  然后从新Generated再次运行还是空白
**原因:** 在themes/文件夹的next主题是空的,因为在我的github上的next主题是作为submodule,是引用别人的仓库,在clone到一台新的机器上时并没有clone到本地来,如果你没有通过submodule引用别人的仓库,就不会出现这个问题.
**解决办法:**
cd F:\Arvin-He.github.io
git submodule init
git submodule update
**注意:** 克隆含有子模块的项目:
克隆一个含有子模块的项目。 当你在克隆这样的项目时，默认会包含该子模块目录，但其中还没有任何文件.
你必须运行两个命令：git submodule init 用来初始化本地配置文件，
而 git submodule update 则从该项目中抓取所有数据并检出父项目中列出的合适的提交。

## 3.6 部署到github上去后发现没有更新,还是上次的页面
**解决办法:**
1. 删除.deploy_git/文件夹<br>
2. hexo clean //清除上次生成的静态文件<br>
3. hexo g //重新生成静态文件<br>
4. hexo d //部署到github上去<br>

## 3.7 点击"标签"和"分类"没有跳出标签和分类

原因:Tags和Category需要手动生成,即需要输入命令生成.
方法:
生成Tags和categories
```
$ cd path/to/hexo
$ hexo new page tags
$ hexo new page categories
$ vim source/tags/index.md 
在date下面一行输入: 
type: "tags"
$ vim source/categories/index.md
在date下面一行输入: 
type: "categories"
```
--- 
# 4. 一些补充

### 4.1 设置标题、分类、标签  
在你的Markdown文章的开头添加, 如果使用hexo new命令新建文章则会自动生成<br>
添加标签和分类,则首先确认站点配置文件里有tag_dir: tags这个配置选项打开
然后确认确认主题配置文件里有tags: /tags这个配置选项打开
最后在你的站点文章中添加如下:
 ```   
---
title: 你的题目
tags: 你的标签
category: 你的分类
---
```
多个标签的设置<br>
方式一：仿照Hexo配置文件中的写法<br>
tags:
  - Hexo
  - HTML
  - JavaScript

方式二：伪JavaScript数组写法<br>
tags: [Hexo,HTML,JavaScript]<br>
多个分类也是如此

### 4.2 设置索引目录里的图片
因为索引设置为提取文档前150个字符，所以想在索引目录中插入图片，就在文章开头插入图片即可。

### 4.3 添加阅读全文的折叠按钮
在你写的文章中你想要折叠的位置输入:
```
<!--More-->
```
然后重新生成部署,就会看到折叠效果了.
