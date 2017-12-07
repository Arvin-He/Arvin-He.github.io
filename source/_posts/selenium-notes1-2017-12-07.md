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


