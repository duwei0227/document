---
layout: post
title: Python获取本机IP地址和Mac地址
categories: [Python]
description: Python获取本机IP地址和Mac地址
permalink: python/ipaddress.html
---


### 1、获取IP地址
此方法在`Windows`和`Linux`下都适用
```python
import socket

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client:
    client.connect(('8.8.8.8', 80))
    ip = client.getsockname()[0]
    
```

### 2、获取MAC地址
目前暂未找到合适的获取mac地址的方法，在一些博客上找到的都是不准确的

暂时采用第三方库 `getmac` 获取
```shell
pip install getmac
```

获取mac地址：
```python
import getmac

getmac.get_mac_address()
```