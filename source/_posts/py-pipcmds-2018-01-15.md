---
title: python之pip常用命令
date: 2018-01-15 17:08:06
tags: python
categories: python
---

## pip常用命令

### 1. 安装第三方库
```
pip install scrapy
python -m pip install scrapy
```

### 2. 更新第三方库
```
pip install scrapy -U
```

### 3. 制作本地*.whl安装离线包
```
pip wheel pycrpyo
```

### 4. 导出pip list列表到requirements.txt
```
pip freeze > requirements.txt
```