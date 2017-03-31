---
title: markdown使用
date: 2017-03-31 22:14:53
tags: markdown
categories: 工具
---

# 1. 表格
语法格式:
```
|head1|head2|head3|
|:----|:----:|-----:|
|content1|content2|content3|
```
其中
* head可有可无
* 冒号用来表示对齐,上面表格分别表示左对齐,居中对齐,右对齐

# 2. 首行缩进
1. 不断行的空白格&nbsp;或&#160;(半个英文空格)<br>
2. 半方大的空白&ensp;或&#8194;(一个英文空格)<br>
3. 全方大的空白&emsp;或&#8195;(两个英文空格)<br>
hello<br>
&nbsp;hello<br>
&ensp;hello<br>
&emsp;hello<br>

# 3. 横隔线
三个星号或者三个减号或者三个下划线即可生成一条横隔线
***
---

# 4.链接和图片
[简书链接](http://www.jianshu.com)

![](http://upload-images.jianshu.io/upload_images/259-0ad0d0bfc1c608b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 5. 引用
在引用的文字前面加上 >, 注：> 和文本之间要保留一个字符的空格。
> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

# 6.粗体和斜体
两个 * 包含一段文本就是粗体的语法，用一个 * 包含一段文本就是斜体的语法.
*一盏灯*， 一片昏黄；**一简书**， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

# 7.代码引用
1. 引用的代码语句只有一段，不分行，可以用 ` 将语句包起来。<br>
2. 引用的代码语句为多行，可以将```置于这段代码的首行和末行。<br>

`hello world!`

```
print(hello world)
print(hello world)
print(hello world)
```

# 8.列表
```
1. 无序列表
2. 有序列表
- 列表1
- 列表2
- 列表3
```
(*注：* 1.和文本之间要保留一个字符的空格。)

# 9.显示链接中带括号的图片
```
![][1]

[1]: http://upload-images.jianshu.io/upload_images/259-0ad0d0bfc1c608b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240

```
![][1]

[1]: http://upload-images.jianshu.io/upload_images/259-0ad0d0bfc1c608b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240

# 10.下划线和删除线
