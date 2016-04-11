---
title: Ubuntu Server 更新源列表
date: 2015-12-15 02:03:42
tags: ubuntu-server
categories: 技术博文
---
<!--
Title: Ubuntu Server 更新源列表
Date: 2015年12月15日
-->
在**Ubuntu Server**下可以使用如下命令更新源列表
```
sudo su #获取root权限
cd /etc/apt
cp source.list source.list.backup #备份源列表
vim source.list #编辑源列表
```
<!-- more -->
在source.list文件中从开头加入下列代码
```
deb http://mirrors.ustc.edu.cn/ubuntu trusty multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu trusty-backports multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu trusty-proposed multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu trusty-security multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu trusty-updates multiverse restricted universe
deb-src http://mirrors.ustc.edu.cn/ubuntu trusty multiverse restricted universe
deb-src http://mirrors.ustc.edu.cn/ubuntu trusty-backports multiverse restricted universe
deb-src http://mirrors.ustc.edu.cn/ubuntu trusty-proposed multiverse restricted universe
deb-src http://mirrors.ustc.edu.cn/ubuntu trusty-security multiverse restricted universe
deb-src http://mirrors.ustc.edu.cn/ubuntu trusty-updates multiverse restricted universe
```

之后使用下列代码更新即可
```
apt-get update
apt-get upgrade -y
```

注：使用vim更新源列表可能用到的复制粘贴命令

- y 复制
- yy 复制整行
- p(小写) 在当前光标后粘贴
- P(大写) 在当前光标粘贴
