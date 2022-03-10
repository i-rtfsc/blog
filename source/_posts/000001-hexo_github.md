---
title: Hexo结合Github搭建博客
comments: true
categories: 
  - Hexo
tags:
  - Hexo
  - 主题
description: "2018年搭建博客记下的笔记"
abbrlink: 6ada37b2
date: 2018-10-06 22:06:58
---

<!--more-->

# 环境搭建

## 安装Node.js
[官网](https://nodejs.org/en/) 安装即可，我们用它来生成静态网页。

## hexo

``` bash
$ sudo npm install -g hexo
```

# 初始化

``` bash
$ mkdir blog
$ cd blog
$ hexo init
$ sudo npm install
$ hexo generate
$ hexo deploy
```
> 这个blog文件夹我们在github上也新建一个仓来管理，方便后续换电脑等情况维护（https://github.com/journeyOS/blog）
> blog下放置的就是我们博客的配置文件，博客的源md文件等等

然后通过 http://localhost:4000 查看本地博客。

# 关联GitHub
- 注册并登陆GitHub账号后，新建仓库，名称必须为user.github.io，如journeyOS.github.io
- 修改_config.yml

``` java
deploy:
  type: git
  repository: https://github.com/journeyOS/journeyOS.github.io.git
  branch: master
```
> https://github.com/journeyOS/journeyOS.github.io.git放置的是生成html后的文件


重新执行
``` bash
$ hexo generate
$ hexo deploy
```
若执行hexo generate出错则执行npm install hexo --save，若执行hexo deploy出错则执行npm install hexo-deployer-git --save。错误修正后再次执行hexo generate和hexo deploy上传到服务器。

# 这篇博客是2018年粗略记的笔记，基本上没啥内容，有需要搭建博客的请参考《搭建hexo、next主题美化》