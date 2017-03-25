---
title: 部署hexo遇到的一些问题
---
## 1. hexo无法上传到github, 本地localhost:4000是可以打开的
On branch master
nothing to commit, working directory clean
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error

解决办法:
deploy:
  type: git
  repository: ssh://git@github.com/Arvin-He/Arvin-He.github.io.git
  branch: master


## 2.hexo部署失败 ERROR Deployer not found: git
解决办法: 在 blog 目录里 执行 npm install hexo-deployer-git --save 

<!--More-->

## 3. Error: fatal: Not a git repository (or any of the parent directories): .git
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

解决办法:
    删除.deploy_git/ 文件夹,然后再次,hexo deploy, 最终就好了.

## 4.一些补充

### 4.1 hexo命令

hexo new "postName" #新建文章<br>
hexo new page "pageName" #新建页面<br>
hexo generate #生成静态页面至public目录<br>
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）<br>
hexo deploy #将.deploy目录部署到GitHub<br>
hexo help  # 查看帮助<br>
hexo version  #查看Hexo的版本<br>
hexo deploy -g  #生成加部署<br>
hexo server -g  #生成加预览<br>
命令的简写<br>
hexo c == hexo clean<br>
hexo n == hexo new<br>
hexo g == hexo generate<br>
hexo s == hexo server<br>
hexo d == hexo deploy<br>

### 4.2 部署文件需要三步    
hexo c<br>
hexo g<br>
hexo d<br>
有时不需要hexo g<br>

### 4.3 设置标题、分类、标签  
在你的Markdown文章的开头添加, 如果使用hexo new命令新建文章则会自动生成<br>
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

tags: [Hexo,HTML,JavaScript]<br>
多个分类也是如此

### 4.4 设置索引目录里的图片
因为索引设置为提取文档前150个字符，所以想在索引目录中插入图片，就在文章开头插入图片即可。

## 5. ERROR Local hexo not found in F:\Arvin-He.github.io
Hexo搭建博客之后用Git已经将所有的source都同步到了git上，在另一台电脑上将源代码clone下来之后，直接执行 hexo g,出现错误<br>
F:\Arvin-He.github.io (source) (hexo-site@0.0.0)<br>
λ hexo g<br>
ERROR Local hexo not found in F:\Arvin-He.github.io<br>
ERROR Try running: 'npm install hexo --save'<br>

解决办法:
cd F:\Arvin-He.github.io
npm install 
hexo g

原因:.gitignore文件里面忽略了node_modules文件夹，就是hexo所依赖的nodejs的模块,所以这个文件夹没有更新上去。所以用npm重新安装即可。

## 6. 执行hexo g或hexo本地测试运行重启后页面空白,提示 : WARN No layout: index.html
此时页面都是白的，使用hexo clean  然后从新Generated再次运行还是空白
原因:在themes/文件夹的next主题是空的,因为next主题是作为submodule,是引用别人的仓库,并没有clone到本地来

解决办法:
cd F:\Arvin-He.github.io
git submodule init
git submodule update


克隆含有子模块的项目:
克隆一个含有子模块的项目。 当你在克隆这样的项目时，默认会包含该子模块目录，但其中还没有任何文件.
你必须运行两个命令：git submodule init 用来初始化本地配置文件，
而 git submodule update 则从该项目中抓取所有数据并检出父项目中列出的合适的提交。