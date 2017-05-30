---
title: Flask笔记
date: 2017-05-28 08:43:19
tags: [Python,Flask]
categories: 编程
---
### Flask路由中使用正则表达式
```python
from werkzeug.routing import BaseConverter

class RegexConverter(BaseConverter):
    def __init__(self, url_map, *item):
        super(RegexConverter, self).__init__(url_map)
        self.regex = item[0]

# 把转换器在初始化时设定到url_map,转换器名称为'regex'
app.url_map.converters['regex'] = RegexConverter


@app.route('/user/<regex("[a-z]{3}"):user_name>')
def user(user_name):
    return 'User %s' % user_name
```

### 多个url指向同一个视图
```python
@app.route('/projects/')
@app.route('/projects2/')
def projects():
    return 'The project page'
```
**注意:**多个路由指向同一视图时,顺序是先从最外层的装饰器路由开始,一旦匹配到路由,就马上执行视图函数,下面的装饰器就不再执行了.

### 使用Manager运行flask应用程序



### 自定义模板过滤器
自定义markdown过滤器
```python
@app.template_fiter('md')
def markdown_to_html(txt):
    from markdown import markdown
    return markdown(txt)

```

### 在模版中调用python函数
```python
@app.context_processor
def inject_methods():
    return dict(read_md=read_md)

def read_md(file_name):
    with open(filename) as md_file:
        content = reduce(lambda x,y: x+y, md_file.readlines())
    return content.decode('utf-8')
    
```
过滤器和全局方法配合使得页面更加灵活

### 模版继承/包含与宏
<hr>  画直线