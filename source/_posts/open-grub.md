---
title: 调出grub并应用其引导Windows+Ubuntu启动
date: 2015-11-25 02:03:42
tags:
    - grub
    - ubuntu
categories: 技术博文
---
<!--
Title: 调出grub并应用其引导Windows+Ubuntu启动
Date: 2015年11月25日
-->
今天一个室友安装 **Windows+Ubuntu** 双系统，碰上一个小问题，现在记录下来。
其实只是一个很简单的问题，但是对于没有用过**linux**系统的少年可能很难，但是有那么句话嘛
> 无折腾不少年

所以只是需要我们多百度查找网上的教程就好了，自己折腾还会记忆深刻呢(^_^)|||
<!-- more -->
## 唠完了，以下是正题

问题是装完**Ubuntu**后重启直接进入了**Ubuntu**，而没有引导界面

## 解决方法
首先打开终端输入以下代码：
```
sudo update-grub
```

输入用户密码后看到以下信息：
```
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-3.16.0-23-generic
Found initrd image: /boot/initrd.img-3.16.0-23-generic
Found memtest86+ image: /boot/memtest86+.elf
Found memtest86+ image: /boot/memtest86+.bin
Found Windows 8 (loader) on /dev/sda1
done
```

发现可以找到windows，那为什么没有**grub**的引导呢
我们输入以下代码看看**grub**的默认配置文件
```
sudo gedit /etc/defauit/grub
```

我们发现**GRUB_HIDDEN_TIMEOUT**项的值为0
将**GRUB_HIDDEN_TIMEOUT=0**改为**GRUB_HIDDEN_TIMEOUT=1**试试

然后再**sudo update-grub**更新grub配置
最后**sudo reboot**重启系统。

结果是出现了**grub**引导且有windows项，问题就这么被我们解决了，想想真的挺简单的。

## 再附上一个修改grub使windows为第一启动项的方法

输入**sudo nautilus**以管理员身份打开文件管理器，图形化界面进入**/etc/grub.d**目录，将文件**30_os-prober**的文件名中的**30**改为**5~10中的一个（不包括5和10）**

然后再**sudo update-grub**更新grub配置
最后**sudo reboot**重启系统。

重启后**Windows 8 (loader) on**就在第一项了，默认进入的也是**windows**
