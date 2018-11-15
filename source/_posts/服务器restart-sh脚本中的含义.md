---
title: 服务器restart.sh脚本中的含义
date: 2018-09-14 17:09:22
tags:
    - 服务器restart.sh脚本中的含义
---
![猫咪老师](/images/页面图片/13.png)
<!--more-->
## Restart.sh 脚本内容：
```
#!/bin/bash
# 重启
echo "restart diditech-iov-unify-webapi-8300 service"

export JAVA_HOME=/maindisk/jdk1.8.0_151
echo ${JAVA_HOME}
dir_base=/app/diditech-iov-unify-webapi/8300
cd ${dir_base}
echo 'deploying...'
jar_name='diditech-iov-unify-webapi.jar'
pid=`ps -ef | grep diditech-iov-unify-webapi | grep 8300 | grep "java" | awk '{print $2}'`
if [ -n "$pid" ]
then
   kill -9 $pid
fi

echo ${dir_base}/${jar_name}
nohup ${JAVA_HOME}/bin/java -jar  ${dir_base}/${jar_name} >/dev/null 2>&1 &
sleep 2
echo 'ok!'

```

## 一．echo命令
Shell的echo指令与PHP的echo指令类似，都是用于字符串的输出。
a)	显示普通字符串
i.	echo “It is a test”    echo It is a test   
这里的双引号完全可以忽略 效果是一样的
```
[rootelocalhost ～]# echo "It is a test"
"It is a test"
[root@localhost ～]# echo It is a test
It is a test
[root@localhost~]#

```

ii.	含有转义字符时 若不加引号 则会丢失信息  “”内信息会完整展示
```
[rootelocalhost 8201]# echo OK!\n OK!n
[rootelocalhost 8201]# echo "OK!\n"
OK!\n

```

b)	显示转移字符
i.	echo "\"It is a test\""   echo \"It is a test\"  
这里引号也是可以省略的
```
[root@localhost ～]# echo "\"It is a test\""
"It is a test"
[rootelocalhost ～]# echo \" It is a test \"
"It is a test "

```

c)	显示变量
先编辑一个test.sh的脚本





