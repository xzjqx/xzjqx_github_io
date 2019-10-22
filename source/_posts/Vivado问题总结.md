---
title: Vivado问题总结
date: 2018-04-14 13:21:46
tags:
categories:
---
记录Vivado使用过程中的一些问题。
<!--more-->
## Vivado 2017.3 启动超时
Vivado启动两分钟后弹出如下界面，显示Vivado启动超时：
![启动超时](启动超时.png)
我是在更新了Windows10系统到1790版本后出现这一问题，在[Xilinx论坛](https://forums.xilinx.com)发现同一问题，Xilinx官方也发现并解决了这个BUG。

> After installing the Windows 10 Fall Creators update (version 1709), when I try to launch Vivado, it fails or crashes.

解决方法如下[AR# 69908](https://www.xilinx.com/support/answers/69908.html)：

> While this update for Windows 10 is not officially supported with Vivado 2017.3, the following work-around is available:
> 1.  Navigate to (Vivado Installed Directory)\2017.3\bin\unwrapped\win64.o
> 2.  Backup '**vivado.exe**' by renaming it to '**vivado.exe.backup**'
> 3.  Copy '**vivado-vg.exe**' and paste it into the same folder.
> 4.  Rename '**vivado-vg - Copy.exe**' to '**vivado.exe**'

简单翻译一下：
当Windows10更新之后发现不能打开Vivado时，执行以下步骤：
- 进入“Vivado安装目录\2017.3\bin\unwrapped\win64.0\”
- 将'**vivado.exe**'重命名为**vivado.exe.backup**'
- 将'**vivado-vg.exe**' 复制到当前目录
- 重命名'**vivado-vg - Copy.exe**'为'**vivado.exe**'
之后再打开Vivado就不会报错了