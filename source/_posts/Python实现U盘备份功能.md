---
title: U盘备份的Python小程序
date: 2016-04-07 00:52:16
tags: python
categories: 学习笔记
---
# 起因
前几天去学校机房做实验，U盘果然又被机房电脑玷污了，又只能格式化了（衰~）
思考解决办法，想着如果每当电脑插入U盘，就自动备份U盘里的文件就好了

# 行动
想到就做吧，正好这几天在学习Python，那就用python来实现这个小脚本吧
<!-- more -->
## Python源码
``` python
import time
import datetime
import os
import os.path
import zipfile
import glob
from zipfile import ZIP_DEFLATED

def GetFileList(dir, fileList):
    newDir = dir
    if os.path.isfile(dir):
        fileList.append(dir)
    elif os.path.isdir(dir):
        for s in os.listdir(dir):
            if s == "System Volume Information":
                continue
            newDir = os.path.join(dir, s)
            GetFileList(newDir, fileList)
    return fileList

def backup(filepath):

    from_dir = []
    # print from_dir
    from_dir = 'G:\\'
    # backup_time
    back_time = time.strftime(u'%Y-%m-%d')

    to_dir = filepath + u':\\backup'
    if not os.path.exists(to_dir):
        os.mkdir(to_dir)
        print u'Completely New Folder.'
    os.chdir(to_dir)

    target = back_time + u'.zip'

    if not os.path.exists(target):
        zip = zipfile.ZipFile(target, 'a', ZIP_DEFLATED)
        file = GetFileList(from_dir, [])
        for f in file:
            zip.write(f)
        print u'Done.'
        zip.close()
    else:
        print u'Having Backups.'

    os.chdir(to_dir)
    list_file = os.listdir(to_dir)

    old_time = datetime.date.today() - datetime.timedelta(30)
    # print str(old_time)[5:7]
    for lis in list_file:
        if lis[5:7] == str(old_time)[5:7]:
            os.remove(lis)


if __name__ == '__main__':

    if os.path.exists('G:') == True:
        backup('D')

```

## 简单解释
上述源码中定义了两个函数：
- GetFileList(dir, fileList): 返回dir下的所有目录和文件
- backup(filepath): 将U盘下的所有文件以日期命名打包，并放在filepath下的backup目录下

## 可改进的地方
本来的想法是当插入U盘是即可自动执行备份操作，但是才知道Windows已经禁用了U盘的自动播放，故没有实现这一功能
后来想到，或许可以在电脑上设计一个定时打开的任务，当它打开时如果电脑插着U盘就备份，这样应该也能做到自动备份的效果，有空再做吧
自己还是太年轻啊，这些很简单的操作都不会[哭脸]

## GitHub
Follow 我的[Python学习项目](https://github.com/xzjqx/python-study)吧，大家一起进步
