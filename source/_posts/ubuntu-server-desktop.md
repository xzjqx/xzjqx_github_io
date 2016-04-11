---
title: Ubuntu Server 安装gnome桌面
date: 2015-12-15 02:03:42
tags: ubuntu-server
categories: 技术博文
---
<!--
Title: Ubuntu Server 安装gnome桌面
Date: 2015年12月15日
-->
上一篇博文提到修改**Ubuntu Server**的源列表到国内服务器中，接下来可以安装gnome桌面来试试

<!--more-->

首先展示几张**ubuntu-gnome-desktop**的图片
![gnome-sock](http://7xo385.com1.z0.glb.clouddn.com/gnome_sock.png)
<!-- more -->
![gnome-login](http://7xo385.com1.z0.glb.clouddn.com/gnome_login.png)
![gnome-desktop](http://7xo385.com1.z0.glb.clouddn.com/gnome_desktop.png)

为防止安装失败首先更新系统软件列表
```
sudo apt-get update
```

执行下列命令安装
```
sudo apt-get install gnome-core
sudo apt-get install ubuntu-gnome-desktop
```

我第一次没有安装**gnome-core**导致，导致多次安装失败，结果还重装了几次系统<del datetime="2015-12-15T09:17:14+00:00">还好是虚拟机</del>
