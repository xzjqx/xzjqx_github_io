---
title: 软件实习笔记一（Qt安装与简单使用）
date: 2015-09-29 01:50:34
tags: Qt5
categories: 学习笔记
---
## 下载带有MinGW的Qt 类库和Qt Creator
我先是下了一个不带MinGW的64位版，相使用自己电脑里的MinGW，但是发现并不能使用，所以，重新下过带MinGW的32位版。
提供<a href='https://www.qt.io/cn/download-open-source/' target="_blank">下载地址</a>和<a href="https://pan.baidu.com/s/1pJOkZjH" target="_blank">网盘地址</a>

<!-- more -->
## 正常安装Qt
安装过程就不多说了，正常的安装过程会把Qt类库和Qt Creator一同装在你的电脑里了

## 简单使用
安装完成后在电脑里找到Qt Creator并打开，打开工具-选项，在左侧找到构建与运行，若安装正确完成，如图所示，在Qt Versions项会自动检测出编译器
![Qt Versions](http://img.blog.csdn.net/20150929203818677)

若不能得到以上画面，那也不要紧，接下来点击“构建套件（Kit）”，修改Desktop志下述画面的情形，再设为默认即可
![构建套件](http://img.blog.csdn.net/20150929203757054)

###  接下来可以开始建立工程简单使用Qt Creator了
点击New Project，选择默认设置
![](http://img.blog.csdn.net/20150929203734994)

再修改工程名，并设置工程目录（**此时要注意路径中不能含有中文，否则无法运行与调试**）
![](http://img.blog.csdn.net/20150929203713543)

接下来一直选择默认设置就好了
![](http://img.blog.csdn.net/20150929203657695)

## 测试
修改main.cpp代码如下：

``` c++
#include<QApplication>  
#include<QVBoxLayout>  
#include<QLabel>  
#include<QPushButton>  

int main(int argc,char *argv[])  
{  
    QApplication app(argc,argv);  

    QWidget *window = new QWidget;  
    window->setWindowTitle("QtTest");  

    //QLabel *label= new QLabel("Hello Qt");  
    QLabel *label = new QLabel("<h2><i>Hello</i>""  <font color = red>Qt</font></h2>");  

    QPushButton *button=new QPushButton("Quit");  
    QObject::connect(button,SIGNAL(clicked()),&app,SLOT(quit()));  

    QVBoxLayout *layout=new QVBoxLayout;  
    layout->addWidget(label);  
    layout->addWidget(button);  
    window->setLayout(layout);  

    window->show();  

    return app.exec();  
}  
```

之后点击运行，会出现下述小窗口，那么恭喜你，你的Qt Creator已经可以完美工作了！！！
![](http://img.blog.csdn.net/20150929203628947)
