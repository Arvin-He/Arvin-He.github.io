---
title: selenium-phatomjs使用
date: 2017-12-11 20:42:16
tags: python
categories: python
---
### 使用selenium和phantomjs爬取动态网页
```python
from selenium import webdriver
url = "http://www.dangniao.com/mh/22996/392953.html"
browser = webdriver.PhantomJS("/usr/local/mysoft/phantomjs-2.1.1/bin/phantomjs")
browser.get(url)
# 模拟用户点击
browser.find_element_by_class_name("zsxiaye").click()
src = browser.find_element_by_css_selector("#wdwailian img").get_attribute("src")
```

### 常用的phantomJS配置选项
```python
from selenium import webdriver
# 引入配置对象DesiredCapabilities
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
dcap = dict(DesiredCapabilities.PHANTOMJS)
#从USER_AGENTS列表中随机选一个浏览器头，伪装浏览器
dcap["phantomjs.page.settings.userAgent"] = (random.choice(USER_AGENTS))
# 不载入图片，爬页面速度会快很多
dcap["phantomjs.page.settings.loadImages"] = False
# 设置代理
service_args = ['--proxy=127.0.0.1:9999','--proxy-type=socks5']
#打开带配置信息的phantomJS浏览器
driver = webdriver.PhantomJS(phantomjs_driver_path, desired_capabilities=dcap,service_args=service_args)                
# 隐式等待5秒，可以自己调节
driver.implicitly_wait(5)
# 设置10秒页面超时返回，类似于requests.get()的timeout选项，driver.get()没有timeout选项
# 以前遇到过driver.get(url)一直不返回，但也不报错的问题，这时程序会卡住，设置超时选项能解决这个问题。
driver.set_page_load_timeout(10)
# 设置10秒脚本超时时间
driver.set_script_timeout(10)
```

### phantomJS的并发问题
phantomJS本身在多线程方面还有很多bug，建议使用多进程,关于多进程，推荐使用multiprocessing库，简洁、高效.
```python
from multiprocessing import Pool
pool = Pool(8)
data_list = pool.map(get, url_list)
pool.close()
pool.join()
```

### phantomJS进程不自动退出问题
主程序退出后，selenium不保证phantomJS也成功退出，最好手动关闭phantomJS进程。
```python
try:
    self.driver.get(url)
    self.wait_()
    return True
except Exception as e:
    # 手动关闭phantomjs进程
    self.driver.quit()
    return False
```



