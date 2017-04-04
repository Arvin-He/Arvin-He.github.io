---
title: maupassant主题使用
date: 2017-04-02 21:36:58
tags: [Hexo]
categories: 工具
---
## 1. maupassant主题简介
Maupassant最初是由Cho为Typecho平台设计开发的一套响应式模板，体积只有20KB，在各种尺寸的设备上表现出色。
[主题地址](https://github.com/tufu9441/maupassant-hexo)

## 2. maupassant主题安装
```
cd 'hexo博客目录'
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
npm install hexo-renderer-jade --save // 渲染器
npm install hexo-renderer-sass --save
npm install hexo-generator-feed --save // rss支持
```
## 3. 修改配置文件
1. Hexo中_config.yml配置文件修改有两处
- language: zh-CN
- theme: maupassant
maupassant已经支持简体中文,以前用next主题时,language: zh-Hans

2. maupassant主题下的_config.yml配置文件修改
```
fancybox: true ## 是否启用Fancybox图片灯箱效果
duoshuo: ## 多说评论
disqus: ## Disqus评论
uyan: ## 友言评论
gentie: ## 网易云跟帖
google_search: false ## google搜索
baidu_search: ## 百度搜索
swiftype: ## swifttypekey站内搜索
tinysou: ## 微搜索
self_search: true ## 使用本地搜索
google_analytics: ## Google Analytics 跟踪ID 
baidu_analytics: ## 百度统计 跟踪ID 
show_category_count: true ## 是否显示侧边栏分类数目
shareto: true ## 是否使用分享按钮
busuanzi: true ## 是否使用卜算子页面访问计数
widgets_on_small_screens: false ## 是否在移动设备屏幕底部显示侧边栏

menu:
  - page: home
    directory: .
    icon: fa-home
  - page: archive
    directory: archives/
    icon: fa-archive
  - page: about
    directory: about/
    icon: fa-user
  - page: commonweal
    directory: 404.html
    icon: fa-heartbeat
  # - page: rss
  #   directory: atom.xml
  #   icon: fa-rss

# 选择和排列希望使用的侧边栏小工具
widgets: ## Six widgets in sidebar provided: search, category, tag, recent_posts, rencent_comments and links.
  - search
  - category
  - tag
  - recent_posts
  - recent_comments
  - links

#  友情链接，请依照格式填写。
links:
  # - title: site-name1
  #   url: http://www.example1.com/
  # - title: site-name2
  #   url: http://www.example2.com/
  # - title: site-name3
  #   url: http://www.example3.com/

timeline:
  - num: 1
    word: 2014/06/12-Start
  - num: 2
    word: 2014/11/29-XXX
  - num: 3
    word: 2015/02/18-DDD
  - num: 4
    word: More

# Static files
# 静态文件存储路径，方便设置CDN缓存
js: js
css: css

# Theme version
# 主题版本，便于静态文件更新后刷新CDN缓存
version: 0.0.0

```

## 4. 生成部署
```
hexo clean
hexo g
hexo s
hexo d
```

## 5. 更换Maupassant遇到的问题
### 5.1 安装好并设置好相应配置文件后,执行hexo clean出现错误
ERROR Plugin load failed: hexo-renderer-sass
Error: %1 is not a valid Win32 application.
原因: hexo-renderer-sass没有安装好
解决办法: 将node_modules文件夹删除,重新安装hexo和Maupassant的依赖,执行npm install
**注意:**执行npm install之前请检查下package.json下的依赖模块是否遗漏,下面是我的package.json.
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

### 5.2 本地运行hexo s出现页面布局错乱
原因:hexo-renderer-sass没有安装好,虽然页面能显示,但是总是错乱.
出现这种情况是因为之前使用的主题是next主题,虽然配置文件都修改好,
在安装hexo-renderer-sass依赖时出了点问题,在hexo-renderer-sass需要python2.7,
而我安装的是python3,安装python2.7,并重新安装hexo-renderer-sass.如果还有问题
就删除node_modules文件夹,重新安装吧.
解决办法: 重新安装hexo-renderer-sass

### 5.3 设置本地搜索 
Maupassant默认使用google搜索,如果要使用本地搜索需要做两件事:
1. 安装依赖模块:hexo-generator-search
```
cd 'hexo博客目录'
npm install -g hexo-generator-search --save
```
2. 修改Maupassant主题的配置文件
将默认的google搜索设置为false,将本地搜索设置true
```
google_search: false ## google搜索
baidu_search:        ## 百度搜索
swiftype:            ## swifttypekey站内搜索
tinysou:             ## 微搜索
self_search: true    ## 使用本地搜索
```