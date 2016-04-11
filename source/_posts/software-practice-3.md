---
title: 软件实习笔记三（Qt5.5连接MySQL5.6）
date: 2015-10-02 02:03:42
tags: Qt5
categories: 学习笔记
---
这段时间一直在尝试使用Qt连接上MySQL数据库，真的是有点难啊！！！
参考了很多网上的资料，最后才成功的连上了数据库，文末将会有我认为有用的链接。
成功后发现其实也没有那么难嘛，只是自己真的是有点傻呢==

<!-- more -->
首先Qt5以后自带MySQL的驱动，不像现在网上那些Qt4的教程，需要我们自己重新编译加载MySQL的驱动，这样整个过程确实简单多了。。。
所以安装MySQL的过程就不赘述了。。。
接下来要介绍连接数据库的各种错误：

### 链接错误
错误显示如下：
![](http://img.blog.csdn.net/20151003003748400)

解决方法：由于在qt中使用了数据库，但是在它的项目文件中却没有相应的说明，所以在工程的.pro文件加入一句代码
```
    QT += sql
```

### 提示能找到MySQL数据库，但是不能加载数据库
错误显示如下：
![](http://img.blog.csdn.net/20151003005433524)

这个问题就比较复杂了，我们需要进行调试，调试过程如下：
- 首先使用qDebug输出错误信息，在main函数中加入以下代码：
```
qDebug() << QSqlDatabase::drivers();  
qDebug() << QCoreApplication::libraryPaths();  
```

应用程序输出显示如下错误信息：
![](http://img.blog.csdn.net/20151003005757654)

发现我们输出的可使用数据库的驱动中存在“**QMYSQL**”，但是在连接时不能找到，这是为什么呢？我们接着向下看，发现在库依赖中的`D:/Qt/Qt5.5.0/5.5/mingw492_32/plugins`目录下存在sqldrivers目录，说明驱动存在，那原因到底是什么呢，再看后面提示的
`D:/build-QtTest-Desktop_Qt_5_5_0_MinGW_32bit-Debug/debug`目录，该目录是程序输入目录，库依赖指到这，说明其中缺少了什么库嘛，那我们现在的目标是去找出缺少的库，加入在这个文件夹中应该就能解决问题了。

- 我们将主函数的代码改成下述代码，再查看程序输出查找原因：
``` c++
#include <qtsql/QSqlDatabase>  
#include <qtsql/QSqlQuery>  
#include <qtsql/QSqlError>  
#include <QPluginLoader>  
#include "mainwindow.h"  

void loadMySqlDriver();  

int main(int argc, char *argv[]) {  
    QCoreApplication app(argc, argv);  
    loadMySqlDriver();  
    return app.exec();  
}  

void loadMySqlDriver() {  
    QPluginLoader loader;  
    // MySQL 驱动插件的路径  
    loader.setFileName("D:/Qt/Qt5.5.0/5.5/mingw492_32/plugins/sqldrivers/qsqlmysqld.dll");  
    qDebug() << loader.load();  
    qDebug() << loader.errorString();  
}  
```

显示错误如下：
![](http://img.blog.csdn.net/20151003011500167)

由此看来加载**MySQL**驱动出错不是找不到驱动插件`qsqlmysqld.dll`，而是找不到`qsqlmysqld.dll`依赖的**DLL**. 把`~/MySQL Server 5.6/lib`下的
`libmysql.dll`和`libmysqld.dll`复制到**exe**文件所在目录(如下图)
![](http://img.blog.csdn.net/20151003011923063)

- 然后再运行程序，这时 MySQL 驱动插件就加载成功了，输出如下：
![](http://img.blog.csdn.net/20151003012133782)

- 最后我们修改main函数查看能否连接数据库：
``` c++
#include <qtsql/QSqlDatabase>  
#include <qtsql/QSqlQuery>  
#include <qtsql/QSqlError>  
#include <QPluginLoader>  
#include "conn.h"  

int main(int argc, char *argv[]) {  
    QCoreApplication app(argc, argv);  
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL");  
    db.setHostName("localhost");  
    db.setDatabaseName("qt");  
    db.setUserName("root");  
    db.setPassword("");  

    if( !db.open() ) {  
        qDebug() << "不能连上数据库";  
    }  
    else {  
        qDebug() << "成功连上数据库";  
    }  

    return app.exec();  
}  
```

- 若输出如下，则成功连接上了数据库：
![](http://img.blog.csdn.net/20151003012346901)

至此，问题彻底解决啦(^_^)

#### 注意事项：
此外，使用过程中还会碰上一些小问题，不要慌，都是可以解决的
比如以下情况：这是因为你的上一个程序没有完全关闭，所以不能打开下一个调试程序，只需在应用程序输出中把之前的程序全部关闭再重新运行就不会出现这个问题了。
![](http://img.blog.csdn.net/20151003012606960)

#### 最后附上参考网站：
<a href="http://blog.csdn.net/lhfeng/article/details/1822784" target="_blank">http://blog.csdn.net/lhfeng/article/details/1822784</a>
<a href="http://blog.chinaunix.net/uid-29278628-id-4049544.html" target="_blank">http://blog.chinaunix.net/uid-29278628-id-4049544.html</a>
<a href="http://tieba.baidu.com/p/3542726621" target="_blank">http://tieba.baidu.com/p/3542726621</a>
<a href="http://qtdebug.com/DB-AccessMySQL.html" target="_blank">http://qtdebug.com/DB-AccessMySQL.html</a>
