---
title: 简单Python爬虫练习
date: 2016-03-27 02:03:42
tags:
    - python
    - 爬虫
categories: 学习笔记
---
<!-- Title: 简单Python爬虫练习 Date: 2016年03月27日 -->
最近开始学习Python的爬虫，在这里做一个记录

首先，我读了一些简单的Python爬虫源码，然后通过入门教程理解代码的意义（这里有一个比较好的入门教程--------[使用 Python 轻松抓取网页](http://wuchong.me/blog/2014/04/24/easy-web-scraping-with-python/)），最后自己动手写了一个小的爬虫程序。
<!-- more -->
我的爬虫测试程序源码：
``` python
#-*-coding:utf8-*-

import requests
import bs4
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

root_url = 'http://acm.tju.edu.cn/toj/'
pageNum = 100
result = ""

for page in range(1001,pageNum+1001):
    index_url = root_url + 'showp%d.html'%(page)
    response = requests.get(index_url)
    soup = bs4.BeautifulSoup(response.text,'lxml')

    problem = soup.select('#problem')[0].get_text()
    fo = open("/mydata/python_test/toj/problem%d"%page, "wb")
    fo.write(problem)

```

这里我记录一下上述代码的意思： 这是一个爬取TOJ网站题目主干的小程序     首先使用requests来加载一个[ web 页面](http://acm.tju.edu.cn/toj/showp1001.html) 查看该网页的源代码中与题目内容有关的部分如下：
``` html
<center>
    <font size="+2">1001.</font> &nbsp;
    <font size="+2" color="blue">Hello, world!</font>
    ...
</center>
...
<div id="problem">
    ...
</div>
```
center标签的前两个font标签分别记录了题目编号和题目标题
div标签的problem类记录了题目内容，我通过 BeautifulSoup 使用 CSS 选择器抽取他们的文本
最终加入problem串中，写入文件

上面就完成了对一个页面题目内容的提取，只需循环处理你想要的页面即可
