---
title: Vue学习笔记1
date: 2017-07-08 15:54:12
tags: [Vue, 前端]
categories: 编程
---


## vue中的一些标签
el: vue的选项/DOM
ol:定义有序列表(order list)。

## vue自带专用属性
指令带有前缀 v-，表示是由 Vue 提供的专用属性。它们会在渲染的 DOM 上产生专门的响应式行为。


条件与循环
v-if="seen"
v-for="todo in todos"

处理用户输入:v-on
v-on:click="reverseMessage"

v-model 指令，使得表单输入和应用程序状态之间的双向绑定变得轻而易举

v-bind:title="message" : 将此元素的 title 属性与 Vue 实例的 message 属性保持关联更新

## 组件
组件本质上是一个被预先定义选项的 Vue 实例
在 Vue 中注册组件:
```javascript
// 定义一个被命名为 todo-item 的新组件
Vue.component('todo-item', {
  template: '<li>这是一个 todo 项</li>'
})
```

在另一个组件模板中组合使用组件:
```html
<ol>
  <!-- 创建一个 todo-item 组件的实例 -->
  <todo-item></todo-item>
</ol>
```

组件与自定义元素之间的关系
Vue 组件非常类似于 Web 组件规范中的自定义元素(Custom Element)