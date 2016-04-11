---
title: 记录Hexo的搭建过程
date: 2015-11-16 11:11:48
tags:
    - Hexo
    - GitHub Pages
    - Coding Pages
categories: 技术博文
---
这是我第一次在自己搭建的博客上编写Markdown文章，现在心里还是有点小激动
经过几天的检索+实验+再检索+再实验，终于把这样的一个Hexo博客搭建好了，现在正是心情大好，是时候LOL一发了……额，别跑题==
<!-- more -->
好了，到正题了，以下是我的搭建实录：
我使用的是 Hexo+Github 部署的博客系统：
> GitHub is a Web-based Git repository hosting service. ——[Wikipedia](https://en.wikipedia.org/wiki/GitHub)
> Hexo is a fast, simple & powerful blog framework, powered by Node.js. ——[Hexo](https://hexo.io/)

# Hexo的安装过程查看[Hexo静态博客使用指南](http://www.tuicool.com/articles/Jva2iaA)
这是在 Mac or Linux 下的安装， Windows 的安装几乎一样，只是要你转到 git bash 下执行这些操作了（~~熟悉使用~~git就能很快上手）

# 配置 Github Pages
使用[Github](https://github.com/)之前需要注册账号，并添加 SSH key，请移步[如何SHH key给GITHUB](http://jingyan.baidu.com/article/a65957f4f0acc624e67f9bc1.html)

# 在 Github 添加仓库
在 Github 中添加名为 yourname.github.io 的仓库，比如我的就是 xiao-jian.github.io 然后将形如 git@github.com:xzjqx/xzjqx.github.io.git 的SSH保存下来方便在 Hexo 中配置

# 配置Hexo
我把网站部署在了 Github 上，修改 Hexo 根目录下的_config.yml文件（在deploy下添加）：

```
deploy:
  type: git
  repository: git@github.com:Xiao-Jian/xiao-jian.github.io.git
  branch: master
```

然后使用如下命令：

```
hexo g #hexo generate
hexo d #hexo deploy
```

第一次部署发现提示错误信息：ERROR Deployer not found : git
需要先运行以下命令npm install hexo-deployer-git --save
再次部署即可成功。

当然还可以在部署之前使用hexo s #hexo serve在本地服务器提前查看效果——在浏览器中输入localhost:4000查看效果
