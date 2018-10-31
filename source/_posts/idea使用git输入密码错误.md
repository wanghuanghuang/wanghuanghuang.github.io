---
title: idea使用git输入密码错误
date: 2018-10-31 09:26:42
tags:
---

今天，使用idea的git下载代码，第一遍输入的密码不对，但是再下载代码时一直报登录信息不正确，却不出现输入密码的地方，上网搜索了一下，得到这两条的解决办法，亲身试用。
1.到git的安装目录下打开CMD,执行命令：
```
git config --system --unset credential.helper
```
![1](/images/idea使用git输入密码错误/1.png)
注意：这样会删除之前所保存的全部账户密码
2.打开控制面板-->管理Windows凭证，选中用到的git网址凭证，点击删除，然后再次git clone 重新输入正确的用户名密码即可。
![2](/images/idea使用git输入密码错误/2.png)
![3](/images/idea使用git输入密码错误/3.png)
![4](/images/idea使用git输入密码错误/4.png)