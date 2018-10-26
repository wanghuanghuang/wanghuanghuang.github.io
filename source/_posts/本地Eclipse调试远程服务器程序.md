---
title: 本地Eclipse调试远程服务器程序
date: 2018-10-23 11:38:58
tags:
---

## 1.	远程服务器：Java程序执行脚本添加配置
![eclipse1](/images/本地Eclipse调试远程服务器程序/eclipse1.png)
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=54321
注意：address指向端口必须不同于原程序端口，让JVM可以区分调试和正常程序。使用docker的程序，需注意将调试端口暴露和映射出来。
[Java远程调试 java -Xdebug各参数说明](https://blog.csdn.net/lantian0802/article/details/40299377)
## 2.	本地：Eclipse配置
![eclipse2](/images/本地Eclipse调试远程服务器程序/eclipse2.png) 
![eclipse3](/images/本地Eclipse调试远程服务器程序/eclipse3.png) 
![eclipse4](/images/本地Eclipse调试远程服务器程序/eclipse4.png) 
## 3.   点击上图Debug按钮开始调试，远程系统的运行会反应在debug pannel和断点上：
![eclipse5](/images/本地Eclipse调试远程服务器程序/eclipse5.png) 
![eclipse6](/images/本地Eclipse调试远程服务器程序/eclipse6.png) 
## 4.   断开远程调试，按Disconnect
![eclipse7](/images/本地Eclipse调试远程服务器程序/eclipse7.png) 
