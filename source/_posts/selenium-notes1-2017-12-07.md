---
title: selenium笔记
date: 2017-12-07 21:39:06
tags: python
categories: python
---
### selenium使用
```python
from selenium import webdriver
driver = webdriver.Firefox()
driver.get("http://www.python.org")
assert "Python" in driver.title
elem = driver.find_element_by_name("q")
elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
assert "No results found." not in driver.page_source
driver.close()
```
### 使用selenium登陆淘宝
```python
# -*- coding:utf-8 -*-
from selenium import webdriver
from pprint import pprint

driver = webdriver.Firefox()
driver.get('https://www.taobao.com/')
assert "淘宝" in driver.title
elem = driver.find_element_by_class_name('member-ft')
tags = elem.find_elements_by_tag_name('a')
login_url = ''
for tag in tags:
    if 'login' in tag.get_attribute('href'):
        login_url = tag.get_attribute('href')

if login_url:
    driver.get(login_url)
    driver.find_element_by_id('J_Quick2Static').click()
    elem = driver.find_element_by_id('TPL_username_1')
    elem.send_keys('淘宝账号')
    elem2 = driver.find_element_by_id('TPL_password_1')
    elem2.send_keys('淘宝账号密码')
    driver.find_element_by_id('J_SubmitStatic').click()
    assert "No results found." not in driver.page_source
    cookies = driver.get_cookies()
    pprint(cookies)
# driver.close()
```
