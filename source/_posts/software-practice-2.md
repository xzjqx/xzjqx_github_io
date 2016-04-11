---
title: 软件实习笔记二（Qt简单对话框实例开发）
date: 2015-09-30 02:01:43
tags: Qt5
categories: 学习笔记
---
最近找到Qt5开发系列教程——《QT5开发及实例》，觉得不错，想着就先按照上面的安排来学习软件开发吧，第一章讲述了一个小的对话框的开发实例。
<!-- more -->
## 首先按照以下过程创建对话框Dialog工程
![](http://img.blog.csdn.net/20150930214127088)
![](http://img.blog.csdn.net/20150930214143294)
![](http://img.blog.csdn.net/20150930214156046)
![](http://img.blog.csdn.net/20150930214205792)

## 使用设计器设计图形化界面
首先打开工程下界面文件中的dialog.ui，进入设计界面如下（**这是我的图形化设计结果**）：
![](http://img.blog.csdn.net/20150930214604064)

其中的“输入半径获得圆面积”、“半径：”、“面积”和面积右边的文本框均是从左边的“**Display Widgets**”拖出来的**Label**类，修改**Label**属性可从右下角的属性栏中进行。“半径：”右侧的文本输入框是**LineEdit**控件，“计算”是**Push Button**控件，同样其属性可以修改。此外还可手动拖动控件或**Dialog**调节其大小。最重要的是“面积：”右侧的Label属性设置，将其中的**QFrame**项设置如下：
![](http://img.blog.csdn.net/20150930215716474)

## 增加按键响应代码实现计算功能
属性全部设置好后就要添加面积计算实现代码了，在dialog.cpp中添加如下代码：
``` java
void Dialog::on_pushButton_clicked()  
{  
    bool ok;  
    QString tmpStr;  
    QString valStr = ui->lineEdit->text();  
    int valInt = valStr.toInt(&ok);  
    double area = valInt * valInt * Pi;  
    ui->areaLable_2->setText(tmpStr.setNum(area));  
}  
```
其中Pi需要全局定义
``` c++
const static double Pi = 3.1415926;
```

## 构建并运行工程检验成果
如果出现下述界面，那么恭喜你完成这这个小程序的开发
![](http://img.blog.csdn.net/20150930220816632)
