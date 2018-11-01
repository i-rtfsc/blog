---
title: Hexo+Github
---

## 环境搭建

### 安装Node.js
[官网](https://nodejs.org/en/) 安装即可，我们用它来生成静态网页。

### hexo

``` bash
$ sudo npm install -g hexo
```

## 初始化

``` bash
$ mkdir blog
$ cd blog
$ hexo init
$ sudo npm install
$ hexo generate
$ hexo deploy
```

然后通过http://localhost:4000查看本地博客。

## 关联GitHub
- 注册并登陆GitHub账号后，新建仓库，名称必须为user.github.io，如journeyOS.github.io
- 修改_config.yml

``` java
deploy:
  type: git
  repository: https://github.com/journeyOS/journeyOS.github.io.git
  branch: master
```
重新执行
``` bash
$ hexo generate
$ hexo deploy
```
若执行hexo generate出错则执行npm install hexo --save，若执行hexo deploy出错则执行npm install hexo-deployer-git --save。错误修正后再次执行hexo generate和hexo deploy上传到服务器。