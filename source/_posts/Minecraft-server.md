---
title: 搭建MineCraft私服
date: 2015-12-23 02:03:42
tags:
    - MineCraft
    - 云服务器
categories: 技术博文
---
<!--
Title: 搭建MineCraft私服
Date: 2015年12月23日
-->
最近购买了腾讯云的云主机，因为腾讯云最近在办活动——[校园云+计划](http://bbs.qcloud.com/thread-9953-1-1.html)，所以我只花了1软妹币/月就购进了一台CPU1核、内存1G、带宽1M、系统盘linux8G的云主机了，这无疑是给我们学生大大的福利啊（这个安利可以忽视）

拿到云主机后我首先做的就是[搭建WordPress博客系统](http://xzjqx.cn/build-wordpress/)，之后才想到搭建一个MineCraft的私人服务器，这样就可以几个小伙伴一起在自己的服务器上愉快的玩耍了。

由于我的服务器上已经有一个WordPress博客系统了，我担心云主机内存不够，所以就在室友买的另一个没用过的服务器上搭建MineCraft私服了。
<!-- more -->
闲话聊完，接下来就是重点了
## 搭建过程简单分为以下几步
- 安装JAVA环境
- MineCraft服务器安装
- MineCraft服务器配置
- MineCraft客户端连接

### **安装JAVA环境**
直接使用命令安装：
```
yum install java
```

安装完成，使用**java -version**查看java版本

### **MineCraft服务器安装**
在[我的网盘](http://pan.baidu.com/s/1jGVXBMm)上下载**Spigot**
密码：d8g2
将下载下来的文件上传到服务器的一个新建目录下，比如**~/minecraft/**
使用下列命令启动**Spigot**
```
java -jar -Xmx512m -Xms512m -XX:MaxPermSize=256M -Dfile.encoding=utf-8 -Duser.timezone=Asia/Hong_Kong ~/minecraft/spigot.jar
```
第一次启动时会自动生成一些配置文件，且会因为一个简单的原因自动关闭服务，这时只需修改生成的**eula.txt**文件，将其中的**eula=false**改为**eula=true**，然后重新启动服务即可。

### **MineCraft服务器配置**
打开~/minecraft/目录下的配置文件**server.properties**，将其中的**online-mode=true**改为**online-mode=false**即可

### **MineCraft客户端连接**
在网上下载同版本的MinCraft客户端，打开后进入多人游戏，选择直接连接，填上自己的主机IP即可连上游戏开始愉快的玩耍了。
