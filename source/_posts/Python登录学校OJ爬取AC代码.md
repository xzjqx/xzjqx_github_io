---
title: Python登录学校OJ爬取AC代码
date: 2016-04-13 22:15:51
tags: python
categories: 学习笔记
---
最近持续学习python中，真是学到深处越来越爱上这门语言了，她“优雅”、“明确”、“简单”的特性深深的吸引了我。
真是太能扯了，屌丝果然只能~~爱上~~语言了.
我在这介绍一部简单易懂的Python教程——[Python2](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013747381369301852037f35874be2b85aa318aad57bda000),[Python3](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

# 起因
之前用Python爬取了Toj的题目主干——[简单Python爬虫练习]()，并以此作为Python爬虫的入门实验，待我Python能力有所长进后，就想到或许可以使用Python把自己之前提交的AC代码全部爬下来，这也许会很有趣。
没错，博主就是天津大学的在校本科生~
<!-- more -->
# 准备
首先我们需要分析一下要爬取的网站架构，这是[TOJ主页](http://acm.tju.edu.cn/toj/)

## 查找AC提交
Toj是靠Run ID来区分所有的提交的，所以我们想要找到自己的AC代码，只需要把自己的AC提交ID记录下来即可
在http://acm.tju.edu.cn/toj/status.php?accept=1&user=xxx 上记录了username为xxx的提交人的AC提交，使用浏览器的审查功能可以看到截图：
![](http://7xo385.com1.z0.glb.clouddn.com/UQA30R%7D%5D9~XMMFKTQ%29%5DBJYK.png)
看上图两个方框圈起来的地方是对应的，这里的Run ID就是我们要找的东西

## 爬取准备
找完所有的AC提交ID，我们就可以开始爬取对应代码了
点击上图中code尺寸的链接就会进入一个密码验证界面——http://acm.tju.edu.cn/toj/show_open.php?sid=1591405
这里的sid就是你打开的代码对应的Run ID
输入密码验证成功后就会进入有AC代码的页面，在审查中打开Network属性可以看到下面的结果：
![](http://7xo385.com1.z0.glb.clouddn.com/@G0W12M~C5~%7D414_LS5%5DQ~8.png)
这里最重要的信息是：
- Request URL:
- user_id:
- sid:1591405
- passwd:
请求的URL为http://acm.tju.edu.cn/toj/show_code.php，表单提交的数据有user_id,sid,passwd
知道这些我们就可以完成上面的密码验证过程了
最后查看该页面的html代码就可以爬下你的AC代码了

# 码代码
## 使用模块
首先看看我用到的所有模块吧
``` python
import getpass #实现不回显输入密码
import urllib  
import urllib2 #这两个是爬虫用的经典库
import bs4     #提取并分析HTML内容的工具
import os      #操作系统相关工具
```

## 获取AC提交ID
按照上面的准备过程，首先我们要找到自己的AC提交ID：
``` python
def getRunId(user_id, sid_url, run_id):
    request = urllib2.Request(sid_url)
    response = urllib2.urlopen(request)
    soup = bs4.BeautifulSoup(response.read(), "lxml")
    tr = soup.select('tr[height="30"]') #按照此属性选择标签
    if len(tr) > 0: #如果标签数不为0，则继续查找下一页
        for x in tr:
            run_id.append(x.get_text()[0:7])
            y = (int)(x.get_text()[0:7])
        sid_url = "http://acm.tju.edu.cn/toj/status.php?user=%s&accept=1&start=%d" %(user_id,y-1)
        getRunId(user_id, sid_url, run_id)
    return run_id
```
这里用到bs4下的BeautifulSoup工具完成了对HTML文档的解析，从中我们找到了所有的AC提交ID

## 获取AC代码
``` python
print 'Printing All Your AC Codes...'
for sid in run_id:
    url = "http://acm.tju.edu.cn/toj/show_code.php"
    values = {
        "user_id" : user_id,
        "sid" : sid,
        "passwd" : passwd
    }
    data = urllib.urlencode(values)
    request = urllib2.Request(url,data)
    response = urllib2.urlopen(request)
    soup = bs4.BeautifulSoup(response.read(), "lxml")
    pid = soup.select('a')[8].get_text()
    code = soup.select('pre')[0].get_text().encode('utf-8')
    fo = open(to_dir + '\\' + pid + u'.cpp', "wb")
    fo.write(code)
```
说明一下，这里模拟了登录验证密码的过程，也就是之前提到的表单提交
与此同时，我们还把AC代码提取出来并保存在本地了，看看下面的截图你就能明白是如何提取的了
![](http://7xo385.com1.z0.glb.clouddn.com/%29%29X39Y@%7DDCR%5BOMREX%25D~2%7B1.png)

## 输入账号密码和代码保存目录
``` python
print 'Please Input Your Username: ',
user_id = raw_input()
passwd = getpass.getpass('Please Input Your Password: ')
print 'Please Input Your Target Directory: ',
to_dir = raw_input()

#Verify Your Username
print 'Verifying Your Username...',
sid_url = "http://acm.tju.edu.cn/toj/status.php?accept=1&user=%s" % (user_id)
request = urllib2.Request(sid_url)
response = urllib2.urlopen(request)
soup = bs4.BeautifulSoup(response.read(), "lxml")
tr = soup.select('tr[height="30"]')
if len(tr) == 0:
    raise Exception ("No Such User")
else:
    print '\nDone!'
    print 'Finding Your AC Run Id...'
    run_id = getRunId(user_id, sid_url, [])
    print 'Done!'

#Verify Your Password
print 'Verifying Your Password...',
url = "http://acm.tju.edu.cn/toj/show_code.php"
sid = tr[0].get_text()[0:7]
values = {
    "user_id" : user_id,
    "sid" : sid,
    "passwd" : passwd
}
data = urllib.urlencode(values)
request = urllib2.Request(url,data)
response = urllib2.urlopen(request)
s = response.read().decode('utf-8', 'ignore')
if s.find("Password Error!") != -1:
    raise Exception ("Password Error!")
else:
    print 'Done!'

#Verify Your to_dir
if os.path.exists(to_dir) is False:
    os.mkdir(to_dir)
```
这里的过程基本上大家都明白了吧，我只是使用了Python自带的raise语句抛出一个错误，达到验证账号密码是否正确的目的

# 完整程序
``` python
# -*- coding: utf-8 -*-
import getpass
import urllib
import urllib2
import bs4
import os

def getRunId(user_id, sid_url, run_id):
    request = urllib2.Request(sid_url)
    response = urllib2.urlopen(request)
    soup = bs4.BeautifulSoup(response.read(), "lxml")
    tr = soup.select('tr[height="30"]')
    if len(tr) > 0:
        for x in tr:
            run_id.append(x.get_text()[0:7])
            y = (int)(x.get_text()[0:7])
        sid_url = "http://acm.tju.edu.cn/toj/status.php?user=%s&accept=1&start=%d" %(user_id,y-1)
        getRunId(user_id, sid_url, run_id)
    return run_id

print 'Please Input Your Username: ',
user_id = raw_input()
passwd = getpass.getpass('Please Input Your Password: ')
print 'Please Input Your Target Directory: ',
to_dir = raw_input()

#Verify Your Username
print 'Verifying Your Username...',
sid_url = "http://acm.tju.edu.cn/toj/status.php?accept=1&user=%s" % (user_id)
request = urllib2.Request(sid_url)
response = urllib2.urlopen(request)
soup = bs4.BeautifulSoup(response.read(), "lxml")
tr = soup.select('tr[height="30"]')
if len(tr) == 0:
    raise Exception ("No Such User")
else:
    print '\nDone!'
    print 'Finding Your AC Run Id...'
    run_id = getRunId(user_id, sid_url, [])
    print 'Done!'

#Verify Your Password
print 'Verifying Your Password...',
url = "http://acm.tju.edu.cn/toj/show_code.php"
sid = tr[0].get_text()[0:7]
values = {
    "user_id" : user_id,
    "sid" : sid,
    "passwd" : passwd
}
data = urllib.urlencode(values)
request = urllib2.Request(url,data)
response = urllib2.urlopen(request)
s = response.read().decode('utf-8', 'ignore')
if s.find("Password Error!") != -1:
    raise Exception ("Password Error!")
else:
    print 'Done!'

#Verify Your to_dir
if os.path.exists(to_dir) is False:
    os.mkdir(to_dir)

print 'Printing All Your AC Codes...'
for sid in run_id:
    url = "http://acm.tju.edu.cn/toj/show_code.php"
    values = {
        "user_id" : user_id,
        "sid" : sid,
        "passwd" : passwd
    }
    data = urllib.urlencode(values)
    request = urllib2.Request(url,data)
    response = urllib2.urlopen(request)
    soup = bs4.BeautifulSoup(response.read(), "lxml")
    pid = soup.select('a')[8].get_text()
    code = soup.select('pre')[0].get_text().encode('utf-8')
    fo = open(to_dir + '\\' + pid + u'.cpp', "wb")
    fo.write(code)
```

## GitHub
Follow 我的[Python学习项目](https://github.com/xzjqx/python-study)吧，大家一起进步
