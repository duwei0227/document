---
layout: post
title: 解决Chrome浏览器默认https访问
categories: [其他]
permalink: other/chrome.html
tags: Chrome
---


### 1、背景
Chrome新版浏览器在访问网址时会默认添加http，对于一部分未支持https或者公司内部的项目来说很不友好

### 2、解决方案
#### 2.1 方案一
在Chrome浏览器地址栏输入以下内容：
```
chrome://net-internals/#hsts
```

![](/img/chrome-url-address.png)

找到 `Delete domain security policies`
在`Domain`栏中输入需要删除的域名后点击`delete`
![](/img/chrome-delete-domain.png)