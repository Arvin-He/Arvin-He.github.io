---
title: Hexo搭建github博客
date: 2017-03-24 15:26:36
tags: [Hexo]
categories: 工具
description: 本文介绍如何采用hexo搭建个人博客,以及遇到的一些问题.
---
![cute cat](hexo-tutorial-2017-03-24/5.jpg)
## 1. 安装Hexo及依赖
### 1.1 准备条件
1. 安装git
2. 有github帐号,并有仓库username.github.io
3. 本地机器保存github的帐号和密码

### 1.2 安装hexo
1. 安装nodejs
2. 安装hexo, npm install -g hexo, 或者 npm install hexo-cli -g

### 1.3 安装hexo依赖
在hexo的根目录下有一个package.json文件,里面是hexo需要的依赖.
如果要添加其他模块的依赖,比如增加本地搜索功能时,需要安装hexo-generator-search模块,则使用命令:
```
npm install -g hexo-generator-search --save
```
在hexo根目录下node_modules文件夹下就会安装hexo-generator-search模块,并在package.json中写入对应的
hexo-generator-search依赖条目.
下面是我的package.json文件所用到的hexo依赖.
```
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.2.2"
  },
  "dependencies": {
    "hexo": "^3.2.2",
    "hexo-deployer-git": "^0.2.0",
    "hexo-generator-archive": "^0.1.4",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-feed": "^1.2.0",
    "hexo-generator-index": "^0.2.0",
    "hexo-generator-json-content": "^3.0.1",
    "hexo-generator-search": "^1.0.4",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.2.0",
    "hexo-renderer-jade": "^0.3.0",
    "hexo-renderer-marked": "^0.2.10",
    "hexo-renderer-sass": "^0.3.1",
    "hexo-renderer-stylus": "^0.3.1",
    "hexo-server": "^0.2.0"
  }
}
```
**注意:** 这些依赖不是全部必须的,根据你个人喜好和所需的功能以及不同主题的不同依赖所决定的,
一开始hexo只安装基本的依赖.例如你安装了hexo-next主题,可查看[hexo-next官方配置教程](http://theme-next.iissnan.com/getting-started.html).

## 2. 配置Hexo
在Hexo中的根目录下有一个\_config.yml配置文件,根据你需要的功能进行配置.
下面我指列出了常见的配置,没有列出的都是默认配置.
```
title: "你的站点名"  #站点名，站点左上角
subtitle: "你站点的副标题"
description: "你站点的描述" #给搜索引擎看的，对站点的描述，可以自定义
author: "站点作者"
language: "语言" # 注意:不同的主题支持的语言配置会不一样,请注意你所选主题支持的语言.如next支持中文语言的配置是zh-Hans,不是zh-CN,当然后期也可能支持.
url: http://username.github.io  # 如果你用到站内搜索,需要配置一下你的url, 否则搜索正常但页面跳转不出来.
tag_dir: tags # 你的标签目录,不打开则点击标签时没反应
archive_dir: archives # 你的归档目录
category_dir: categories # 你的分类目录
per_page: 10 # 配置你一页显示文章数目,默认是10篇
# 设置站内检索功能
search:
  path: search.xml
  field: all
theme: next # 主题设置,根据你所选主题,在这里设置你的主题,没配置的话,就是空白也
# 部署
deploy:
  type: git
  repository: https://github.com/username/username.github.io.git # 根据你的github帐号的usename来替换
  branch: master
```

## 3. 常用的功能配置
### 3.1 设置Tags(标签)页面
**注意:** Tags目录不会自动生成,需要你先配置再手动生成.
```
$ cd "hexo目录"
$ hexo new page tags              # 会在source/tags/目录下生成index.md
$ vim source/tags/index.md 
在你的文章的开头即front-matter中添加一项:
type: "tags"
```
1. 也就是首先要确认站点配置文件里有tag_dir: tags这个配置选项打开, 
然后确认你所选主题配置文件里有tags: /tags这个配置选项打开(假如你选择next主题的话)
2. 使用hexo new命令新建文章则会自动生成标签目录
3. 在你的文章的开头即front-matter中添加一项type: "tags",添加如下:
```   
---
title: 你的题目
tags: 你的标签
category: 你的分类
---
```
多个标签的设置
方式一：仿照Hexo配置文件中的写法
tags:
  - Hexo
  - HTML
  - JavaScript

方式二：伪JavaScript数组写法
tags: [Hexo,HTML,JavaScript]

### 3.2 设置categories(分类)页面
```
$ cd "hexo目录"
$ hexo new page categories        # 会在source/categories/目录下生成index.md
$ vim source/categories/index.md
在你的文章的开头即front-matter中添加一项:
type: "categories"
```
1. 也就是首先要确认站点配置文件里有category_dir: categories这个配置选项打开, 
然后确认你所选主题配置文件里有category_dir: /category_dir这个配置选项打开(假如你选择next主题的话)
2. 使用hexo new命令新建文章则会自动生成分类目录
3. 在你的文章的开头即front-matter中添加一项type: "categories",添加如下:
```   
---
title: 你的题目
tags: 你的标签
category: 你的分类
---
```
多个分类设置
方式一：仿照Hexo配置文件中的写法
categories:
  - Hexo
  - HTML
  - JavaScript

方式二：伪JavaScript数组写法
categories: [Hexo,HTML,JavaScript]

### 3.3 设置about页面
```
$ cd path/to/hexo
$ hexo new page "about"              # 会在source/about/目录下生成index.md
```
然后在souce/about/index.md中写入你想表达的内容.

### 3.4 设置索引目录里的图片
因为索引设置为提取文档前150个字符，所以想在索引目录中插入图片，就在文章开头插入图片即可。

### 3.5 文章摘要
首页默认显示文章摘要而非全文，可以在文章的front-matter中填写一项description:来设置你想显示的摘要，
或者直接在文章内容中插入<\!--more-->以隐藏后面的内容。
```
<!--More-->
```
若两者都未设置，则自动截取文章第一段作为摘要。然后重新生成部署,就会看到折叠效果了.

### 3.6 使用hexo站内搜索
1. 安装 hexo-generator-search，在站点的根目录下执行以下命令：
```
$ npm install hexo-generator-search --save
```
2. 站点配置文件_config.yml配置
**注意:**在**站点**下的_config.yml配置,不是主题里的_config.yml.
```
search:
  path: search.xml
  field: all
```
3. 主题_config.yml配置
本地搜索设置为true,否则界面上不会显示搜索按钮
```
# Local search
local_search:
  enable: true
```

### 3.7 在文章中插入本地图片
1. 配置hexo下的\_config.yml中post_asset_folder: true
2. 安装CodeFalling/hexo-asset-image的插件来加载本地图片,[插件地址](https://github.com/CodeFalling/hexo-asset-image)
```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```
3. 插入图片
![cute cat](newfolder/5.jpg)
**注意:**当设置post_asset_folder为true参数后，在Hexo new newfile时会自动建立一个与文章同名的文件夹，
可以把与该文章相关的所有资源都放到那个文件夹，如此一来，便可以方便的管理使用资源。

## 4. 使用hexo
新建一个目录如:'hexo目录'
cd 'hexo目录'
hexo init       //文件夹自动生成建网站所需的文件
npm install     //在文件夹下安装node_modules,即安装依赖
hexo g          //生成静态文件
hexo s          //本地运行
hexo d          //部署到github上去

## 5. hexo命令详解
### 5.1 hexo命令
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

## 6. 搭建hexo过程中遇到的问题
### 6.1 hexo无法上传到github, 但在本地localhost:4000是可以打开的
On branch master
nothing to commit, working directory clean
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
原因:本地没有保存github用户名和密码,因为hexo d,是直接将生成的静态文件部署到github上去,一步到位的,
意思就是部署的时候就要能拿到github的用户名和密码,如果拿不到就报错,部署不上去.
**注意:** 使用的是https协议,因为当没有在本地保存你的github帐号的用户名
和密码时,使用https协议push内容到github上去时就要每次手动输入github帐号上用户名和密码.
而hexo d部署是一步到位的,中间没有机会让你输入你的github账户用户名和密码.
解决办法:
1. 使用https协议时,本地保存你的github账户的用户名和密码
deploy:
  type: git
  repository: https://github.com/yourusername/yourusername.github.io.git
  branch: master<br>
2. 使用ssh协议,本地生成ssh key,并将公匙放到你的github上
deploy:
  type: git
  repository: ssh://git@github.com/yourusername/yourusername.github.io.git
  branch: master

### 6.2 hexo部署失败 ERROR Deployer not found: git
原因: 缺少nodejs的依赖
解决办法: 在'hexo目录'的目录下执行 npm install hexo-deployer-git --save 

### 6.3 Error: fatal: Not a git repository (or any of the parent directories): .git
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

### 6.4 ERROR Local hexo not found in F:\username.github.io
Hexo搭建博客之后用Git已经将所有的source都同步到了git上，在另一台电脑上将源代码clone下来之后，直接执行hexo g,出现错误.
$ hexo g<
ERROR Local hexo not found
ERROR Try running: 'npm install hexo --save'
**原因:** 在另外一台机器上clone下来的内容,缺少hexo的依赖,因为在.gitignore里忽略将node_nodules文件夹push到github上去.就是hexo所依赖的nodejs的模块,所以用npm重新安装即可.
**解决办法:**
cd 'hexo目录'
npm install 
hexo g

### 6.5 执行hexo g或hexo本地测试运行重启后页面空白,提示 : WARN No layout: index.html
此时页面都是白的，使用hexo clean  然后从新Generated再次运行还是空白
**原因:** 在themes/文件夹的next主题是空的,因为在我的github上的next主题是作为submodule,是引用别人的仓库,在clone到一台新的机器上时并没有clone到本地来,如果你没有通过submodule引用别人的仓库,就不会出现这个问题.
**解决办法:**
cd 'hexo目录'
git submodule init
git submodule update
**注意:** 克隆含有子模块的项目:
克隆一个含有子模块的项目。 当你在克隆这样的项目时，默认会包含该子模块目录，但其中还没有任何文件.
你必须运行两个命令：git submodule init 用来初始化本地配置文件，
而 git submodule update 则从该项目中抓取所有数据并检出父项目中列出的合适的提交。

### 6.6 部署到github上去后发现没有更新,还是上次的页面
**解决办法:**
1. 删除.deploy_git/文件夹<br>
2. hexo clean //清除上次生成的静态文件<br>
3. hexo g //重新生成静态文件<br>
4. hexo d //部署到github上去<br>

### 6.7 点击"标签"和"分类"没有跳出标签和分类
原因:Tags和Category需要手动生成,即需要输入命令生成.
方法:
生成Tags和categories
```
$ cd path/to/hexo
$ hexo new page tags              # 会在source/tags/目录下生成index.md
$ hexo new page categories         # 会在source/categories/目录下生成index.md
$ vim source/tags/index.md 
在date下面一行输入: 
type: "tags"
$ vim source/categories/index.md
在date下面一行输入: 
type: "categories"
```

### 6.8 关于404页面在本地正常显示,在Github上不显示问题
原因:Github Pages强制要求https，所以文档内对js和css的请求都需要经过https传输的才行，而腾讯的404公益页面使用的默认为http.
其中search_children.js主要提取了data.js及page.js两个文件，前者是寻找儿童的数据，在Github中没问题,
后者中默认都是用http加载的js和css，所以不能直接用，故修改为https方式获取js与css，直接将page.js内容加入404.html页面，
具体内容详见我的Github上的404页面.
[知乎上解决办法链接](https://www.zhihu.com/question/49153090/answer/146374701)

### 6.9 hexo使用hexo-generator-search站内搜索
问题: 可以搜索。但是搜索的页面跳转不对.
原因: 站点下的_config.yml配置文件的url没有配置.
解决: 将url设置你github.io的站点的url.
```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://username.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```