---
title: Appium笔记
date: 2017-12-08 14:22:04
tags: node
categories: node
---


### 关于appium
手机app自动化测试，现有支持的app的手机平台（Andriod和IOS）, 选择Appium工具。因为Andriod和IOS，Appium都支持。

web自动化测试的路线是这样的：
编程语言基础--->测试框架--->webdriver API（selenium2）--->开发自动化测试项目。

移动自动化的测试的路线则是这样的：
编程语言基础--->测试框架--->android/IOS开发测试基础--->appium API --->开发移动自动化项目。

appium就是node的其中一个开源项目，appiun server端是用node实现，遵循了REST架构，所以appium可以用node的包管理工具npm来进行安装。

### 安装appium
1. 使用淘宝镜像安装, 输入"cnpm install -g appium" 即可在线安装.
2. 安装appium的客户端，基于python的开发环境，可以用pip安装appium客户端, 输入"pip install Appium-Python-Client"，