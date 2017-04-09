---
title: python练手：简单的爬虫爬取贴吧内容
date: 2015-02-22 00:03:39
tags:
  - Python
---

（顺便做了寒假作业）

需求：

1.  爬取指定贴吧第一页所有帖子内容
2.  简单的去除内容中的图片标签
用python写一个简单爬虫应该是最初级的东西吧= =

设计思路：get指定链接的html内容，正则表达式获取所有帖子链接，get链接内容html，处理链接内容，写入文本文件（打印）。

首先导入需要的模块,设置两个全局变量表示要爬的站点（贴吧）

```python
#coding=utf-8
import httplib2
import re

site = "http://tieba.baidu.com"
bing = "http://tieba.baidu.com/f?kw=%E6%B9%96%E5%8D%97%E5%A4%A7%E5%AD%A6&amp;fr=index&amp;fp=0&amp;ie=utf-8"
```
  
（此处为湖南大学吧）

<!--more-->

第一个函数：获取首页html，返回string字符串。

```python
def getHtml(url):
    h = httplib2.Http(".cache")
    (resp, page) = h.request(url,"GET")
    return page.decode()
```

函数：获取html中帖子链接

首先分析贴吧页面结构

[![QQ图片20150221153838](http://blogr.hcyue.ml/wp-content/uploads/2015/02/QQ图片20150221153838-1024x348.png)](http://blogr.hcyue.ml/wp-content/uploads/2015/02/QQ图片20150221153838.png)

&nbsp;

每个帖子链接指向类似/p/&lt;id&gt;的绝对链接。

于是编写函数用正则表达式获取链接地址：

```python
def getLink(self):
    reg = r'href="(/p/\d*)"'
    return re.findall(reg, self)
```

然后分析帖子页面

[![QQ图片20150221155504](http://blogr.hcyue.ml/wp-content/uploads/2015/02/QQ图片20150221155504-1024x460.png)](http://blogr.hcyue.ml/wp-content/uploads/2015/02/QQ图片20150221155504.png)

每一楼的内容部分由最外层的&lt;cc&gt;包括，然后是一个带有唯一数字id和通用class的div。

所以编写函数用正则表达式筛选

```python
def getContent(self):
    reg = r'&lt;div id="post_content_\d*" class="d_post_content j_d_post_content "&gt;\
            (.*)&lt;/div&gt;&lt;br&gt;&lt;/cc&gt;'
    return re.findall(reg,self)
```

返回每一楼的内容的list。

然后是对提取的内容进行整理，去除html标签，简单排版。

```python
def cleanContext(text):
    text = re.sub(r'&lt;br&gt;', '\r\n', text)
    text = re.sub(r'&lt;.*&gt;', '', text)
    return text
```

最后是调用蜘蛛的函数。

```python
def spaider(url):
    html = getHtml(url)
    link_list = getLink(html)

    with open('a.txt', 'wt') as f:
        i=1
        for link in link_list:
            page = getHtml(site+link)
            h = getContent(page)
            for text in h:
                text = cleanContext(text)
                f.write(text+'\r\n')
            f.write('\r\n')
            print('writing',i,'th\n')
            i+=1</code></pre>
```

先获取首页，再逐个链接获取筛选，写入a.txt。

最后全部源码：

```python
#coding=utf-8
import httplib2
import re

site = "http://tieba.baidu.com"
bing = "http://tieba.baidu.com/f?kw=%E6%B9%96%E5%8D%97%E5%A4%A7%E5%AD%A6&amp;fr=index&amp;fp=0&amp;ie=utf-8"

def getHtml(url):
    h = httplib2.Http(".cache")
    (resp, page) = h.request(url,"GET")
    return page.decode()

def getLink(self):
    reg = r'href="(/p/\d*)"'
    return re.findall(reg, self)

def getContent(self):
    reg = r'&lt;div id="post_content_\d*" class="d_post_content j_d_post_content "&gt;(.*)&lt;/div&gt;&lt;br&gt;&lt;/cc&gt;'
    return re.findall(reg,self)

def cleanContext(text):
    text = re.sub(r'&lt;br&gt;', '\r\n', text)
    text = re.sub(r'&lt;.*&gt;', '', text)
    return text

def spaider(url):
    html = getHtml(url)
    link_list = getLink(html)

    with open('a.txt', 'wt') as f:
        i=1
        for link in link_list:
            page = getHtml(site+link)
            h = getContent(page)
            for text in h:
                text = cleanContext(text)
                f.write(text+'\r\n')
            f.write('\r\n')
            print('writing',i,'th\n')
            i+=1

spaider(bing)
```
&nbsp;

执行结果（a.txt）：

[![QQ截图20150221172533](http://blogr.hcyue.ml/wp-content/uploads/2015/02/QQ截图20150221172533.jpg)](http://blogr.hcyue.ml/wp-content/uploads/2015/02/QQ截图20150221172533.jpg)

看起来还不错wwwww

&nbsp;