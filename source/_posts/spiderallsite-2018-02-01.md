---
title: 全站爬虫
date: 2018-02-01 11:14:30
tags: 爬虫
categories: python
---
## 全站爬虫简介
python version: 3.6, 爬取全站url,并保存html到mongodb.

## 爬虫功能
* 生产者消费者模式
* 支持协程并发
* 爬取深度可配置
* 爬取总量可配置
* 自动过滤无关URL

## 相关代码段
allsitepider.py
```python
# -*- coding:utf-8 -*-
import os
import time
import chardet
import requests
import requests.exceptions
from urllib.parse import urlparse
from bs4 import BeautifulSoup
import asyncio
import traceback

try:
    from .config import get_header, func_timer, connect_mongo
    from .config import TIMEOUT, DEPTH, MONGODB, AMOUNTS, CONTENT_LENGTH, DELAY, QUEUE_SIZE
    from .logger import Logging
except ImportError:
    from config import get_header, func_timer, connect_mongo
    from config import TIMEOUT, DEPTH, MONGODB, AMOUNTS, CONTENT_LENGTH, DELAY, QUEUE_SIZE
    from logger import Logging

_logger = Logging(os.path.splitext(os.path.basename(__file__))[0]).logObject


class Url(object):
    url = ''
    end_type = 'WEB'
    parent_url = ''
    depth = 0
    def __init__(self, url, end_type, parent_url_obj=None):
        self.url = url
        self.end_type = end_type
        if parent_url_obj:
            self.parent_url = parent_url_obj.url
            self.depth = parent_url_obj.depth + 1


class AllSiteSpider(object):

    def __init__(self):
        self.client = connect_mongo(MONGODB)
        self.movie = self.client[MONGODB['collection']]
        self.filter_set = set()
        self.q = asyncio.Queue(maxsize=QUEUE_SIZE)
        self.consume_num = 0
        self.produce_num = 0
        self.stop_extract_links = False

    def start_crawl(self, start_url, end_type):
        self.domain = getattr(urlparse(start_url), 'netloc', '')
        self.end_type = end_type
        url = start_url.strip()
        if url.endswith('/'):
            url = url[:-1]
        self.filter_set.add(url)
        start_url_obj = Url(url, self.end_type)
        self.q.put_nowait(start_url_obj)
        event_loop = asyncio.get_event_loop()
        _logger.info('开始全站爬取:start_url=[{}], Depth={}, end_type={}, parent_url=None'.format(
            start_url_obj.url, start_url_obj.depth, start_url_obj.end_type))
        try:
            event_loop.run_until_complete(self.run(event_loop))
        finally:
            event_loop.close()

    async def run(self, loop):
        consumers = [loop.create_task(self.consumer(i)) for i in range(5)]
        await asyncio.wait(consumers)

    async def consumer(self, num):
        while True:
            try:
                url_obj = await self.q.get()
                self.consume_num += 1
                _logger.info('消费者{}:开始消耗第 {} 个, QSize={}, Depth={}, URL=[{}]'.format(
                    num, self.consume_num, self.q.qsize(), url_obj.depth, url_obj.url))
                self.q.task_done()
                await self.producer(url_obj)
            finally:
                if self.q.empty():
                    _logger.info('队列已空,任务结束,退出EventLoop.')
                    break
        await self.q.join()

    async def producer(self, url_obj):
        self.produce_num += 1
        _logger.info('生产者:开始生产第 {} 个, QSize={}, Depth={}, URL=[{}]'.format(
            self.produce_num, self.q.qsize(), url_obj.depth, url_obj.url))
        try:
            response = self.download_html(url_obj.url)
            self.save_html(response)
            await asyncio.sleep(DELAY)
            if url_obj.depth < DEPTH:
                if not self.stop_extract_links:
                    links = self.extract_links(response, url_obj)
                    if (self.consume_num+self.q.qsize()) < AMOUNTS:
                        for url_obj in links:
                            await self.q.put(url_obj)
                        _logger.info('向当前队列Put {} 个url, 当前队列QSize={}'.format(len(links) if links else 0, self.q.qsize()))
                    else:
                        _logger.info('当前已消耗URL + 当前队列QSize为{},大于全站爬取最大限制数目{},停止向队列Put URL,丢弃提取的 {} 个url,并停止解析提取之后的所有页面链接'.format(
                            self.consume_num+self.q.qsize(), AMOUNTS, len(links) if links else 0))
                        self.stop_extract_links = True
            else:
                _logger.warning('当前URL深度为 {}, 达到最大深度限制,不再提取页面URL,当前URL=[{}]'.format(url_obj.depth, url_obj.url))
            _logger.info('当前全局set共有 {} 个url'.format(len(self.filter_set)))
        except Exception as e:
            _logger.error('生产者异常:{}'.format(traceback.format_exc()))

    def download_html(self, url, retry=2):
        _logger.info('下载页面:开始下载HTML页面...')
        try:
            response = requests.get(url, headers=get_header(self.end_type), stream=True, timeout=TIMEOUT)
            if int(response.headers.get('Content-Length', 0)) < CONTENT_LENGTH:
                response.encoding = chardet.detect(response.content)['encoding']
                return response
        except requests.exceptions.RequestException as e:
            _logger.error('访问链接失败: {}, ERROR: {}'.format(url, e))
            if retry > 0:
                _logger.info('链接: {} 准备重试倒数第 {} 次'.format(url, retry+1))
                self.download_html(url, retry-1)

    def save_html(self, response):
        doc = {}
        if response and response.status_code not in range(400, 600):
            doc['request'] = {
                'url': response.url,
                'headers': response.request.headers,
                'method': response.request.method
            }
            doc['response'] = {
                'headers': response.headers,
                'content': response.content,
                'status_code': response.status_code
            }
            doc['end_type'] = self.end_type
            doc['domain'] = getattr(urlparse(response.url), 'netloc', '')
            doc['insert_time'] = format((time.time() * 1000), '0.0f')

        if doc:
            _logger.info('保存数据:解析并保存数据到MONGODB数据库...')
            self.movie.insert(doc)
        else:
            if response:
                _logger.warning('没有解析到数据, status_code={}, url={}'.format(
                    response.status_code, response.url))
            else:
                _logger.warning('没有解析到数据')

    def record_urls(self, filename, url):
        with open(filename, 'a', encoding='utf-8') as f:
            f.write(url + '\n')

    def filter_url(self, url, parent_url_obj, filtered_num, record=False):
        url_obj = None
        new_url = url.strip()
        if new_url.endswith('/'):
            new_url = new_url[:-1]
        if not self.filter_set.__contains__(new_url) and self.domain in new_url:
            self.filter_set.add(new_url)
            url_obj = Url(new_url, self.end_type, parent_url_obj)
            if record:
                self.record_urls('unique.txt', new_url)
        else:
            filtered_num += 1
            if record:
                self.record_urls('filtered.txt', new_url)
        return url_obj, filtered_num

    def extract_links(self, response, url_obj):
        if response:
            link_list = []
            filtered_num = 0
            parent_url_obj = url_obj
            parent_url = parent_url_obj.url
            soup = BeautifulSoup(response.text, 'lxml')
            tags = soup.find_all(True)
            for tag in tags:
                url = tag.get('href', None)
                if url is not None:
                    if url.startswith('javascript') or url.startswith('index'):
                        continue
                    elif url.startswith('//'):
                        url = '{}:{}'.format(getattr(urlparse(parent_url), 'scheme', ''), url)
                    elif url.startswith('/'):
                        url = '{}://{}{}'.format(getattr(urlparse(parent_url), 'scheme', ''),
                                                 getattr(urlparse(parent_url), 'netloc', ''), url)
                    url_obj, filtered_num = self.filter_url(url, parent_url_obj, filtered_num)
                    if url_obj:
                        link_list.append(url_obj)
            _logger.info('提取Urls:该页面共提取 {} 个, 过滤 {} 个'.format(len(link_list), filtered_num))
            return link_list
        else:
            _logger.error("该页面为None")


# if __name__ == '__main__':
#     # start_url = 'http://www.tongfudun.com'
#     # start_url = 'https://www.baidu.com'
#     # start_url = 'http://www.tcnet.com.cn/'
#     start_url = 'https://www.douban.com'
#     allsiteSpider = AllSiteSpider()
#     allsiteSpider.start_crawl(start_url, 'WEB')
```

### 配置文件
config.py
```python
# -*- coding:utf-8 -*-
import random
import time
from functools import wraps
from pymongo import MongoClient

# 爬取深度
DEPTH = 3
# 单个HTML的Content-Length为1M
CONTENT_LENGTH = 1*1024*1024
# 整站爬取总量
AMOUNTS = 500
# 网络请求超时
TIMEOUT = 10
# 爬取延时
DELAY = 1
# 队列大小
QUEUE_SIZE = 500
# 域名允许控制
ALLOWED_DOMAINS = [

]
# MONGODB参数
MONGODB = {
    "user": "Goshawk",
    "passwd": "yYZmrjJ8xauAe#t7",
    "host": "192.168.1.141:27017",
    "dbname": "Goshawk",
    "collection": "my_mainurls"
}
# WEB端user-agent代理头
WEB_USER_AGENTS = [
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Acoo Browser; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)",
    "Mozilla/4.0 (compatible; MSIE 7.0; AOL 9.5; AOLBuild 4337.35; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
    "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
    "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
    "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
    "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
    "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.11 TaoBrowser/2.0 Safari/536.11",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.71 Safari/537.1 LBBROWSER",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E; LBBROWSER)",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 732; .NET4.0C; .NET4.0E; LBBROWSER)",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.84 Safari/535.11 LBBROWSER",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E; QQBrowser/7.0.3698.400)",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 732; .NET4.0C; .NET4.0E)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; SV1; QQDownload 732; .NET4.0C; .NET4.0E; 360SE)",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 732; .NET4.0C; .NET4.0E)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
    "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.89 Safari/537.1",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.89 Safari/537.1",
    "Mozilla/5.0 (iPad; U; CPU OS 4_2_1 like Mac OS X; zh-cn) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8C148 Safari/6533.18.5",
    "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b13pre) Gecko/20110307 Firefox/4.0b13pre",
    "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:16.0) Gecko/20100101 Firefox/16.0",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11",
    "Mozilla/5.0 (X11; U; Linux x86_64; zh-CN; rv:1.9.2.10) Gecko/20100922 Ubuntu/10.10 (maverick) Firefox/3.6.10"
]
# 移动端user-agent代理头
MOBILE_USER_AGENTS = [

]

def get_header(end_type):
    """
    :param end_type: WEB/MOBILE
    :return: header
    """
    if end_type == 'WEB':
        user_agent = random.choice(WEB_USER_AGENTS)
    else:
        user_agent = random.choice(MOBILE_USER_AGENTS)
    return {
        'User-Agent': user_agent,
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Connection': 'keep-alive',
        'Accept-Encoding': 'gzip, deflate'}


def func_timer(function):
    @wraps(function)
    def function_timer(*args, **kwargs):
        print('[Function: {name} start...]'.format(name = function.__name__))
        t0 = time.time()
        result = function(*args, **kwargs)
        t1 = time.time()
        print('[Function: {name} finished, spent time: {time:.2f}s]'.format(name = function.__name__,time = t1 - t0))
        return result
    return function_timer


def connect_mongo(MONGODB):
    """
    :param MONGODB: dict type
    :return: db object
    """
    client = MongoClient(
        'mongodb://{}:{}@{}/{}'.format(MONGODB['user'],
                                       MONGODB['passwd'],
                                       MONGODB['host'],
                                       MONGODB['dbname']))
    return client[MONGODB['dbname']]
```

### 日志模块
logger.py
```python
# -*- coding: utf-8 -*-
import os
import logging
import logging.handlers

LOG_FORMAT = "%(asctime)s [%(levelname)s] [%(filename)s] [line:%(lineno)d]: %(message)s"
DATA_LOG_FORMAT = "%m/%d/%Y %H:%M:%S"


# 使用root_logger
def init_root_logger_settings(logConsole=True):
    formatter = logging.Formatter(fmt=LOG_FORMAT, datefmt=DATA_LOG_FORMAT)
    log_dir = os.path.join(os.path.dirname(os.path.abspath(__file__)), "logs")
    if not os.path.exists(log_dir):
        os.mkdir(log_dir)
    logging.getLogger().setLevel(logging.INFO)
    log_name = os.path.join(log_dir, __name__.replace('_', '') if '_' in __name__ else __name__)
    # 文件输出日志
    fh = logging.handlers.TimedRotatingFileHandler(
        filename=log_name, when='midnight', interval=1, encoding='utf-8')
    fh.setLevel(logging.INFO)
    fh.suffix = "%Y-%m-%d.log"
    fh.setFormatter(formatter)
    logging.getLogger().addHandler(fh)
    # 控制台输出日志
    if logConsole:
        ch = logging.StreamHandler()
        ch.setLevel(logging.DEBUG)
        ch.setFormatter(formatter)
        logging.getLogger().addHandler(ch)

      
# 生成指定名称的logger
class Logging(object):

    def __init__(self, log_name=None, log_level="info", console=True):
        log_dir = os.path.join(os.path.dirname((os.path.dirname(os.path.abspath(__file__)))), "logs")
        if not os.path.exists(log_dir):
            os.mkdir(log_dir)
        if log_name is None:
            _log_name = os.path.join(log_dir, __name__.replace('_', '') if '_' in __name__ else __name__)
        else:
            _log_name = os.path.join(log_dir, log_name)
        self.logObject = logging.getLogger(_log_name)
        self._set_level(log_level)
        # 文件输出日志
        fh = logging.handlers.TimedRotatingFileHandler(filename=_log_name,
                                                       when='midnight',
                                                       interval=1,
                                                       encoding='utf-8')
        fh.suffix = "%Y-%m-%d.log"
        formatter = logging.Formatter(fmt=LOG_FORMAT, datefmt=DATA_LOG_FORMAT)
        fh.setLevel(logging.INFO)
        fh.setFormatter(formatter)
        self.logObject.addHandler(fh)
        # 控制台输出日志
        if console:
            ch = logging.StreamHandler()
            ch.setLevel(logging.DEBUG)
            ch.setFormatter(formatter)
            self.logObject.addHandler(ch)

    def _set_level(self, log_level):
        if log_level == "warning":
            self.logObject.setLevel(logging.WARNING)
        elif log_level == "debug":
            self.logObject.setLevel(logging.DEBUG)
        elif log_level == "error":
            self.logObject.setLevel(logging.ERROR)
        elif log_level == "critical":
            self.logObject.setLevel(logging.CRITICAL)
        else:
            self.logObject.setLevel(logging.INFO)
```
