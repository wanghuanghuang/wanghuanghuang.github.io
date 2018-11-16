---
title: 用命令行把项目部署到github
date: 2018-09-14 17:01:54
tags:
    - 用命令行把项目部署到github
---
![猫咪老师](/images/页面图片/16.png)
## 1.项目首次提交到github上
a)	打开项目所在文件位置，右击点击Git Bash Here.
b)	$git init 		//初始化
c)	$git commit –m ‘第一次提交’	//提交，并注释信息“第一次提交”
d)	$git remote add origin git@github.com:{github用户名}/{repository名}.git	//链接远程github项目
e)	$git push –u origin master 	//将本地项目提交到GitHub项目上去
## 2.删除GitHub上的文件
a)	打开项目所在的文件位置，右击点击git bash here
b)	$git pull origin master 	//将远程仓库上的项目拉取下来
c)	$dir	 	//查看有哪些文件夹
d)	$git rm –r –cached 文件名		//删除某文件
e)	$git commit –m ‘删除说明’		//提交，添加说明
f)	$git push –u origin master 		//将本次更改提交到GitHub项目上去
注意：
本地项目中的文件不会被删除，不会影响本地文件操作。删除的只是远程仓库中的文件。
每次增加文件或删除文件，都要commit然后直接git push –u origin master,就可以同步到GitHub上了。

