---
title: 本地IDEA调试远程服务器程序
date: 2018-10-23 15:48:53
tags:
---
![猫咪老师](/images/页面图片/15.png)
<!--more-->
## 服务端配置
首先，我们要让要让远程服务器支持远程调试功能，在项目启动项上追加特定的 JVM 参数即可，参数如下：
•   晚于 JDK 1.4.X 版本
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=4001
•	JDK 1.4.X 版本
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=4001
•	JDK 1.3.X 或早起版本
-Xnoagent -Djava.compiler=NONE -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=4001
大家根据不同的 JDK 版本使用不同的启动参数即可。本人使用的是 Tomcat 7 + Java 7 ，在 catalina.bat（linux 环境设置 catalina.sh）中设置 JAVA_OPTS 节点即可，“address=4001” 中的 “4001”为调试端口，大家可以根据需求自行设置（友情提示：不要占用网站的端口，有可能会导致项目启动报错）。
## IDE 配置
然后，配置一下客户端就可以啦，Idea 的客户端配置非常简单在如下图所示的位置：
![1](/images/本地IDEA调试远程服务器程序/1.png)
点击 Edit Configurations，即可进入 Run/Debug Configurations 界面:
![2](/images/本地IDEA调试远程服务器程序/2.png)
点击左上角的“+”标识，可以在下拉框中发现“Remote”选项，选择确定后，出现右侧区域，首先在HOST（标注2）框中配置需要调试的远程服务器地址，然后在调试端口狂（标注3）中调试端口号（这里的端口号和服务器端的端口号保持一致，本示例中是“4001”），点击 OK 即可。
## 调试步骤
最后，进入调试流程，整个项目的调试也非常简单，点击下图中的 debug 按钮，当 console 窗口中打印出 “Connected to the target VM, address: 127.0.0.1:4001', transport: 'socket'”即表示链接成功：
![3](/images/本地IDEA调试远程服务器程序/3.png)
当然啦，也会有不顺利的情况出现，如：
•	服务器端口限制，比如服务器屏蔽了“4001”端口，会导致远程调试失败；
•	本地代码和远程代码不一致，也会导致远程代码调试失败；
