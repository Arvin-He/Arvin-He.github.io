---
title: Flask模板
date: 2017-04-05 11:07:17
tags: flask
categories: python
---

![](flask-module-2017-04-03/5.jpg)

### 1. jinjia2模版分隔符
jinjia2模版有两种分隔符
1. `{`% ... %`}` # 用于执行类似 for 循环或者赋值的声明，
2. `{`{ ... }`}` # 用于输出表达的结果到模板中

### 2. 如何组织模版
模板/目录结构反映应用程序的 URL 结构
```
templates/
    layout.html
    index.html
    about.html
    profile/
        layout.html
        index.html
    photos.html
    admin/
        layout.html
        index.html
        analytics.html
```

### 3. 模版继承
通常有一个顶层的 layout.html，它定义了应用程序的通用布局以及网站的每一部分。
如果看看上面的目录的话，会看到一个顶层的 myapp/templates/layout.html，
同样还有 myapp/templates/profile/layout.html 和 
myapp/templates/admin/layout.html。最后两个文件继承和修改第一个文件。
继承是用 `{`% extends %`}` 和 `{`% block %`}` 标签实现的。

在父模板中，我们能定义由子模板来填充的块。
```
{# _myapp/templates/layout.html_ #}
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
    {% block body %}
        <h1>This heading is defined in the parent.</h1>
    {% endblock %}
    </body>
</html>
```

子模版
```
{# _myapp/templates/index.html_ #}
{% extends "layout.html" %}
{% block title %}Hello world!{% endblock %}
{% block body %}
    {{ super() }}
    <h2>This heading is defined in the child.</h2>
{% endblock %}
```
super() 函数让我们渲染父级块的内容。

### 4. 创建宏
宏就像由模板代码构成的函数。
抽象出重复出现的代码片段到宏,以减少大量的重复代码.
在应用程序导航的HTML上，需要给一个 “活跃的”链接一个 class(class=”active”)。
没有宏的话，要编写一大段 if ... else 语句检查每一个链接来找到正处于活跃的一个。
宏提供了一种模块化代码的方式；像函数一样工作.
1. 如何使用宏标记一个活跃的链接
```
{# myapp/templates/layout.html #}
{% from "macros.html" import nav_link with context %}
<!DOCTYPE html>
<html lang="en">
    <head>
    {% block head %}
        <title>My application</title>
    {% endblock %}
    </head>
    <body>
        <ul class="nav-list">
            {{ nav_link('home', 'Home') }}
            {{ nav_link('about', 'About') }}
            {{ nav_link('contact', 'Get in touch') }}
        </ul>
    {% block body %}
    {% endblock %}
    </body>
</html>
```
在模板中就是调用一个未定义的宏 - nav_link,
并向其传递两个参数：目标端点（例如，目标视图的函数名）以及要显示的文本。

2. 如何定义宏
定义在模板中使用的 nav_link 宏
```
{# myapp/templates/macros.html #}

{% macro nav_link(endpoint, text) %}
{% if request.endpoint.endswith(endpoint) %}
    <li class="active"><a href="{{ url_for(endpoint) }}">{{text}}</a></li>
{% else %}
    <li><a href="{{ url_for(endpoint) }}">{{text}}</a></li>
{% endif %}
{% endmacro %}
```
**注意:**导入语句中指定了 with context。
Jinja 的 context 是由传递到 render_template() 函数的参数以及来自Python代码的Jinja环境上下文组成。
对于模板来说，这些变量在模板被渲染的时候是可用的。
一些变量是明显地由我们传入，例如，render_template("index.html", color="red")，
但是还有一些变量和函数是由 Flask 自动地包含在上下文中，例如，request, g 和 session。
当我们说{`% from ... import ... with context %`}的时候，就是告诉 Jinja 这些变量对宏也可用。

### 5. 自定义过滤器
过滤器就是由Python代码组成的函数并且能在模板中使用
Jinja 过滤器是一个函数，它能够在{`{ ... }`}中用于处理一个表达式的结果。在表达式结果输出到模板之前它就被调用。
```
<h2>{{ article.title|title }}</h2>
```
在这段代码中，title 过滤器接收 article.title 作为参数并且返回一个过滤后的标题，接着过滤后的标题将会输出到模板中。
这就像 UNIX 的“管道化”一个程序到另一个程序的输出。

现在要在myapp/util/filters.py 中定义我们的过滤器。这里给出一个 util 包，它里面有各种各样的模块。
```
# myapp/util/filters.py

from .. import app

@app.template_filter()
def caps(text):
    """Convert a string to all caps."""
    return text.uppercase()
```
在这段代码中我们使用 @app.template_filter() 装饰器注册我们的函数成一个 Jinja 过滤器。

默认的过滤器名称就是函数的名称，但是你可以传入一个参数到装饰器中来改变它。
```
@app.template_filter('make_caps')
def caps(text):
    """Convert a string to all caps."""
    return text.uppercase()
```
现在我们可以在模板中调用 make_caps 而不是caps：{`{ "hello world!"|make_caps }`}。
为了要让我们的过滤器在模板中可用的话，我们只需要在我们的顶层 \__init.py__ 的中导入它。
```
# myapp/__init__.py

# Make sure app has been initialized first to prevent circular imports.
from .util import filters
```