---
title: UnknownHostException
date: 2019-05-10 16:30:53
tags:
---

springboot 工程，各服务之间调用，两个服务均已注册到erueka上，服务之间调用报错：UnknownHostException：
###解决：
修改hosts文件
1、window环境：

hosts文件位置：C:\windows\system32\drivers\etc\hosts

修改如图：
ip + 域名
![host文件修改](source/images/UnknownHostException/host文件修改.png)

刷新方式：

ctrl+r，输入CMD，回车

在命令行执行:ipconfig /flushdns     #清除DNS缓存内容。
ps:ipconfig /displaydns    //显示DNS缓存内容

host文件内容：
```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost

192.168.1.105 7d86cee55bf5
```

2、linux环境

文件位置：/etc/hosts

刷新命令：systemctl restart nscd
