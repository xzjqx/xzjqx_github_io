---
title: 使用Sublime直接编译运行C/C++源文件
date: 2015-12-27 02:03:42
tags:
    - Sublime
    - C/C++
categories: 技术博文
---
<!--
Title: 使用Sublime直接编译运行C/C++源文件
Date: 2015年12月27日
-->
以前只知道Sublime是一个有个性的编辑器，最近才发现原来不仅仅是有个性的编译器。
今天就讲讲怎么直接在Sublime上编译执行C/C++源程序
<!-- more -->
## **C语言**
在菜单栏点击Tools->Build System->New Build System会打开一个名为**untitled.sublime-build**的文件，将下段代码复制进该文件
```
{
    "working_dir": "$file_path",
    "cmd": "gcc -Wall $file_name -o $file_base_name",
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
    "selector": "source.c",

    "variants":
    [
        {   
        "name": "Run",
            "shell_cmd": "gcc -Wall $file -o $file_base_name && start cmd /c \"${file_path}/${file_base_name} & pause\""
        }
    ]
}
```
然后按`Ctrl+S`保存，打开一个User文件夹，说明保存为用户设置，可以修改名称为**c.sublime-build**
最后在Tools->Build Sysrem下选择c选项即可

**使用**
打开一个c源文件，按下组合键`Ctrl+B`，第一次会弹出一个复选框，其中有`c`或`c - run`两项，点击第一项为编译，第二项为编译执行

**注意**
组合键`Ctrl+B`为打开上一次选择的编译选项
组合键`Ctrl+Shift+B`为打开选择框，可选两种编译选项

## **C++**
c++的配置与上述一致，只需将代码修改如下：
```
{
    "encoding": "utf-8",
    "working_dir": "$file_path",
    "shell_cmd": "g++ -Wall -std=c++0x $file_name -o $file_base_name",
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
    "selector": "source.c++",

    "variants":
    [
        {   
        "name": "Run",
            "shell_cmd": "g++ -Wall -std=c++0x $file -o $file_base_name && start cmd /c \"${file_path}/${file_base_name} & pause\""
        }
    ]
}
```
再将名字改为c++.sublime-build即可
