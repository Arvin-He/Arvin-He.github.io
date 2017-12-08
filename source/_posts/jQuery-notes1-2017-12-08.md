---
title: jQuery笔记
date: 2017-12-08 17:31:58
tags: js
categories: js
---
### jQuery介绍
jQuery是JavaScript世界中使用最广泛的一个库。
jQuery能帮我们干这些事情：
* 消除浏览器差异：你不需要自己写冗长的代码来针对不同的浏览器来绑定事件，编写AJAX等代码；
* 简洁的操作DOM的方法：写$('#test')肯定比document.getElementById('test')来得简洁；
* 轻松实现动画、修改CSS等各种操作。
jQuery的理念“Write Less, Do More“，让你写更少的代码，完成更多的工作！

目前jQuery有1.x和2.x两个主要版本，区别在于2.x移除了对古老的IE 6、7、8的支持，
因此2.x的代码更精简。选择哪个版本主要取决于你是否想支持IE 6~8。
jQuery只是一个jquery-xxx.js文件，但你会看到有compressed（已压缩）和uncompressed（未压缩）两种版本，使用时完全一样，
但如果你想深入研究jQuery源码，那就用uncompressed版本。

### jQuery使用
$是著名的jQuery符号。jQuery把所有功能全部封装在一个全局变量jQuery中，而$也是一个合法的变量名，它是变量jQuery的别名：
```javascript
window.jQuery; // jQuery(selector, context)
window.$; // jQuery(selector, context)
$ === jQuery; // true
typeof($); // 'function'
```
$本质上就是一个函数，但是函数也是对象，于是$除了可以直接调用外，也可以有很多其他属性。

**注意:** 你看到的$函数名可能不是jQuery(selector, context)，因为很多JavaScript压缩工具可以对函数名和参数改名，
压缩过的jQuery源码$函数可能变成a(b, c)。这也被叫做代码混淆,也起到一定的安全作用.

绝大多数时候，直接用$,但是，如果$这个变量被占用了，而且还不能改，那我们就只能让jQuery把$变量交出来，然后就只能使用jQuery这个变量.

### 选择器
选择器是jQuery的核心。一个选择器写出来类似$('#dom-id')。
Query的选择器就是帮助我们快速定位到一个或多个DOM节点。
```JavaScript
// 按ID查找,注意: #abc以#开头。返回的对象是jQuery对象。
// 查找<div id="abc">:
var div = $('#abc');

// 按tag查找
var ps = $('p'); // 返回所有<p>节点
ps.length; // 数一数页面有多少个<p>节点

// 按class查找, 按class查找注意在class名称前加一个.
var a = $('.red'); // 所有节点包含`class="red"`都将返回
// 例如:
// <div class="red">...</div>
// <p class="green red">...</p>
// 节点有多个class，我们可以查找同时包含red和green的节点：
var a = $('.red.green'); // 注意没有空格！
// 符合条件的节点：
// <div class="red green">...</div>
// <div class="blue green red">...</div>

// 按属性查找
var email = $('[name=email]'); // 找出<??? name="email">
var icons = $('[name^=icon]'); // 找出所有name属性值以icon开头的DOM
var names = $('[name$=with]'); // 找出所有name属性值以with结尾的DOM
var icons = $('[class^="icon-"]'); // 找出所有class包含至少一个以`icon-`开头的DOM
```


组合查找
`var emailInput = $('input[name=email]'); // 不会找出<div name="email">`
`var tr = $('tr.red'); // 找出<tr class="red ...">...</tr>`

多项选择器
多项选择器就是把多个选择器用,组合起来一块选：
```JavaScript
$('p,div'); // 把<p>和<div>都选出来
$('p.red,p.green'); // 把<p class="red">和<p class="green">都选出来
```

总之jQuery的选择器不会返回undefined或者null，这样的好处是你不必在下一行判断if (div === undefined)。
jQuery对象和DOM对象之间可以互相转化：
```javascript
var div = $('#abc'); // jQuery对象
var divDom = div.get(0); // 假设存在div，获取第1个DOM元素
var another = $(divDom); // 重新把DOM包装为jQuery对象
```

通常情况下你不需要获取DOM对象，直接使用jQuery对象更加方便。
如果你拿到了一个DOM对象，那可以简单地调用$(aDomObject)把它变成jQuery对象，这样就可以方便地使用jQuery的API了。


