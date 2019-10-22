---
title: Calculator实现文档
date: 2018-06-12 22:27:54
tags:
	- C/C++
	- Qt
categories: Virtual Studio
---

### 项目介绍

本项目在Virtual Studio2015中实现了一个类似Windows系统中的计算器程序，使用VS中的QT Virtual Studio Tools工具在VS中调用Qt的库函数，以通过Qt实现这样一个计算器。 

### 代码结构

#### 整体架构

Calculator首先通过Qt Designer完成计算器界面的设计，再利用信号和槽函数将计算器中的按键和特定函数相连，当点击计算器按键时，触发槽函数，完成数据的录入，进一步完成表达式的计算，打印出结果。

Calculator类图如下所示：

![Calculator UML](https://github.com/xzjqx/Calculator_Qt/blob/master/images/Calculator_UML.jpg) 

#### UI设计

通过Qt Designer完成计算器UI的设计，在Qt Designer中，通过拖拽控件完成界面设计。

界面最上方是两个QLabel--expression和result，expression显示当前输入的表达式，result显示计算结果；其余控件全部都是QPushButton，分别对应数字键和操作符。

#### 信号与槽函数

Calculator用到的所有信号均为QPushButton的clicked()信号，这个信号当button被按下时发出，触发对应的槽函数。使用Qt Designer的图形化界面连接信号与槽函数。

Calculator共实现了五个槽函数：

- getValue()：所有的数字按键加上小数点与这个槽函数相连，从按键中获取输入的数字；
- getExpr()：所有的计算操作符与这个槽函数相连，从按键中判断进行何种计算，同时这个槽函数还会完成计算的任务；
- clear()：清除键“c”与这个槽函数相连，完成清除所有数据的操作；
- del()：退格键“<-”与这个槽函数相连，完成清除数字最后一位的操作；
- equal()：等于键“=”与这个槽函数相连，完成计算结果并打印的操作。

#### 异常处理

Calculator进行了两种运算过程中异常的判断——除0异常和根号下负数异常。实现方法都是在进行相应运算前，使用函数判断数据是否合法，不合法的情况下抛出异常，判断函数的实现如下： 

```C++
double isZero(double v) {
    if (v == 0)
        throw v;
    return v;
}

double isNeg(double v) {
    if (v < 0)
        throw v;
    return v;
}
```

在运算前调用判断函数确定异常的过程如下：

```C++
try {
    a = isZero(a);
    num = b / a;
}
catch (double) {
    clear();
    isExcept = true;
    ui.result->setText("Invalid input");
}
try {
    value = isNeg(value);
    value = sqrt(value);
    val_tmp = "sqrt(" + val_tmp + ")";
}
catch (double) {
    clear();
    isExcept = true;
    ui.result->setText("Invalid input");
}
```



### 算法

Calculator使用的算法主要集中在getExpr()槽函数对输入的计算上，主要维护两个栈operandStack和operatorStack来进行计算，算法如下：

 

Phase1：点击数字键，调用getValue()槽函数进行数字的记录，重复该步骤直到输入一个操作符；

Phase2：点击运算键，获取输入的运算符，调用getExpr()槽函数

- 如果上一个点击的button是数字键，将记录的数据压入operandStack；
- 如果运算符长度为1（也就是双目运算符）：
  - 如果操作符是“(”，将“(”压入operatorStack；
  - 如果操作符是“)”，重复计算栈中的数据，直到遇到一个“(”；
  - 如果运算符是“+”或“-”，重复计算栈中的所有数据，并把操作符压入operatorStack；
  - 如果运算符是“x”或“/”，重复计算栈中的所有的乘除运算，并把操作符压入operatorStack；
- 如果运算符长度大于1（也就是单目运算符），则计算当前单目运算，并把结果压入operandStack；

Phase3：点击等号键时，将栈中剩余部分全部弹出比计算结果，打印最终结果

### 效果展示

![](https://github.com/xzjqx/Calculator_Qt/blob/master/images/operator_priority.gif)

![](https://github.com/xzjqx/Calculator_Qt/blob/master/images/parentheses.gif)

![](https://github.com/xzjqx/Calculator_Qt/blob/master/images/unary_operation.gif)

![](https://github.com/xzjqx/Calculator_Qt/blob/master/images/exception.gif)


