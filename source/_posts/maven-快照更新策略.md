---
title: maven 快照更新策略
date: 2019-09-04 14:29:03
tags:
---

maven快照更新策略：
依赖每次改动时都会自动带上时间戳，编译时，会检查依赖的时间戳，若是晚于本地仓库依赖的时间戳，就会进行更新。
快照更新并不是每次install都会自动更新，更新策略：每日更新、永远检查更新、从不检查更新和自定义时间间隔更新，默认的是每天更新一次。
如果想要时时更新，可以操作：
1.更改setting文件，如图
![更改setting文件](/images/maven 快照更新策略/setting.png)
2.idea设置，如图
![idea设置](/images/maven 快照更新策略/idea.jpg)
3.命令设置
maven clean install -U
如jenkins设置发布时命令，如图
![jenkins设置](/images/maven 快照更新策略/jenkins.png)
