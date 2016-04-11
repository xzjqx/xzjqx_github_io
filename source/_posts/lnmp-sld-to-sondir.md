---
title: LNMP中二级域名指向网站子目录
date: 2015-12-29 02:03:42
tags:
    - LNMP
    - 二级域名
categories: 技术博文
---
<!--
Title: LNMP中二级域名指向网站子目录
date: 2015年12月29日
-->
最近真的比较闲啊，又开始折腾域名二服务器的事情了，下面就是过程


## **二级域名**
用下面这个例子帮助我们清晰的分清楚顶级域名、以及域名、二级域名：

- .com顶级域名
- baidu .com 一级域名
- www.baidu .com 二级域名
- bbs.baidu .com 二级域名
- tieba.baidu .com 二级域名
<!-- more -->
## **网站子目录**
我们可能会希望在自己的服务器上搭建多个网站，这时候如果把所有的网站文件都放在一个目录下，可能会造成一些冲突或~~强迫症~~？（我是后者多一点）
那这个时候我们就希望在一个文件夹里保存一个网站了，这就需要一个网站子目录了。

可是这时候又有一个新的问题了，当我们想访问这个子目录是或许会是这样**xxxx.cn/xx**或是IP/xx这样
不管怎样都要输入繁琐的子目录名，这样是不是很麻烦。但是如果能让二级域名指向这个就行了，比如使用xx.xxxx.cn访问该子目录的网站，这样是不是很方便了

## **二级域名指向子目录**
现在我们知道用二级域名指向子目录的重要性了吧。
由于我是使用LNMP一键安装包安装的，所以下面的方法也是建立在LNMP上的，不一样的请右转百度。

首先在你的域名服务商添加一条解析记录，使用A记录即可，我的如下：
![A记录](http://7xo385.com1.z0.glb.clouddn.com/ng_A.png)

之后就需要在服务器端做修改了：
首先进入nginx的配置文件目录
```
cd /usr/local/nginx/conf/vhost
```

如果之前已经通过LNMP搭好了自己的主域名网站（即使用vhost.sh脚本设置域名及默认文件夹，具体可看我的这篇文章[云服务器搭建WordPress个人博客系统](http://xzjqx.cn/2015/12/23/build-wordpress/)），这时**vhost**目录下会存在一个以“默认一级域名.conf”为名称的配置文件，我的就是**xzjqx.cn.conf**

这时仿照一级域名的配置文件，我们新建一个二级域名的配置文件，我的例子如下：
文件名：ng.xzjqx.cn.conf
内容：
```
server
{
    listen 80;
    server_name ng.xzjqx.cn;
    index index.html index.htm index.php;
    root /home/wwwroot/default/nginx;
}
```
其中server_name后填入你的二级域名，root后填入网站子目录

最后重启您先服务：
```
/etc/init.d/nginx restart
```

这样我们就能开开心心的使用指向二级域名访问网站子目录了
这是我上面所用的例子：[ng.xzjqx.cn](ng.xzjqx.cn)
这是LNMP安装完成后的默认主页
