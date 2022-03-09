---
title: Linux开机自启动服务
categories:
  - Linux
tags:
  - Linux
  - Service
description: Linux开机自启动VPN服务
comments: true
abbrlink: 87884557
date: 2021-11-11 22:17:11
---
<!--more-->
<meta name="referrer" content="no-referrer"/>


# 背景
liunx上要用翻墙还是比较费劲的，当然如果你能忍受蓝灯这种龟速的网络就当我没说。因为我用的服务商推荐用clash，当然用ssr这些也行。但既然推荐是clash，咱还是尝试着linux上装clash，总的来说还是比较麻烦的。为了防止后续换系统忘记也好，或者是分享给需要的朋友也好，这里还是做一个简单的记录。

# 下载
clash正式的发行版，是通过github上下载的。对应的地址：[https://github.com/Dreamacro/clash/releases](https://github.com/Dreamacro/clash/releases)
目前最新版本是1.9.1，下载之后允许bin文件是会发生段错误的。我一路降级到1.6.0才能正常运行。这是真坑啊，差点放弃了。

# 运行
如果只是简单的使用，下载之后运行bin文件，然后再把配置文件改成自己服务商的配置即可。默认的配置目录在home目录下的~/.config/clash/
```
drwxr-xr-x  2 solo solo 4.0K 1月  20 15:16 .
drwxr-xr-x 30 solo solo 4.0K 1月  20 15:07 ..
-rw-r--r--  1 solo solo  16K 1月  19 10:12 cache.db
-rw-r--r--  1 solo solo  57K 1月  20 15:15 config.yaml
-rw-r--r--  1 solo solo 4.0M 1月  20 15:18 Country.mmdb
```
更新好config.yaml、Country.mmdb之后再打开[http://clash.razord.top/#/settings](http://clash.razord.top/#/settings)更改配置。
![clash_setting.png](https://cdn.nlark.com/yuque/0/2022/png/1759879/1642670363380-989e1e27-fd76-4d1a-9acb-105c3da60a8d.png#clientId=u6b02b9ef-b1c0-4&from=ui&id=u3111df5e&margin=%5Bobject%20Object%5D&name=clash_setting.png&originHeight=615&originWidth=1906&originalType=binary&ratio=1&size=76351&status=done&style=none&taskId=u7c66b74b-f8ab-4a21-a8c8-272b5963bdd)

# linux系统配置
![vpn_setting.png](https://cdn.nlark.com/yuque/0/2022/png/1759879/1642670884200-0e4d14a9-5c75-4e5e-98af-555c5c2e174b.png#clientId=u6b02b9ef-b1c0-4&from=ui&id=uaca87408&margin=%5Bobject%20Object%5D&name=vpn_setting.png&originHeight=784&originWidth=790&originalType=binary&ratio=1&size=80236&status=done&style=none&taskId=u87a9c6bc-cb01-4ef1-944d-523d4639270)
端口号对应上就可以了。

# 开机自启动
配置一个服务，在/etc/systemd/system/下创建clash.service文件
```
sudo vim /etc/systemd/system/clash.service
```
把以下内容复制到clash.service
```
[Unit] 
Description=clash daemon
[Service] 
Type=simple 
User=root 
ExecStart=/home/solo/bin/clash -d /home/solo/bin/config/clash/
Restart=on-failure  
[Install] 
WantedBy=multi-user.target
```
其中：ExecStart后面带上需要执行的脚本或者bin文件，这里多加-d /home/solo/bin/config/clash是因为我指定其配置文件在/home/solo/bin/config/clash目录。
最后通过systemctl命令reload服务，启用服务，启动服务即可。
```
sudo systemctl daemon-reload 
sudo systemctl enable clash
sudo systemctl start clash
sudo systemctl status clash
```
 
# 终端代理命令
export https_proxy=[http://127.0.0.1:](http://127.0.0.1:7890)4780 http_proxy=[http://127.0.0.1:](http://127.0.0.1:7890)4780 all_proxy=socks5://127.0.0.1:4781

# （额外）机器人服务

- sudo vim /etc/systemd/system/bot.service
```
[Unit] 
Description=bot daemon
[Service] 
Type=simple 
User=root 
ExecStart=/home/solo/miniconda3/envs/py36tf1.15/bin/python /home/solo/ext-data/code/github/global_scripts/bot_gerrit_review_schedule.py
Restart=on-failure  
[Install] 
WantedBy=multi-user.target
```

- sudo systemctl daemon-reload & sudo systemctl enable bot
- sudo systemctl restart bot 
- sudo systemctl status bot
