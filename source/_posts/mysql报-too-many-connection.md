---
title: mysql报 too many connection
date: 2019-05-21 09:07:40
tags:
---

too many connection -mysql连接数过多  
查询现有哪些连接方法：
1.选择需要查询的database，新建查询
2.命令 show PROCESSLIST
连接结果如图：
![picture1](source/images/mysql报too many connection/show_processList.png)
拷贝出到文本里 筛选自己账户下的sleep状态的线程ID kill掉 就ok了。


批量kill方案
1) 方案1：通过SQL语句临时文件
```
# 构造出批量kill的sql文件tmp
mysql>select concat('KILL ',id,';') from information_schema.processlist where user='root' into outfile '/tmp/a.txt';
Query OK, 2 rows affected (0.00 sec)
# 执行该文件tmp 
mysql>source /tmp/a.txt;
Query OK, 0 rows affected (0.00 sec)
```
2) 方案2：通过Shell脚本实现
```
#杀掉锁定的MySQL连接
for id in `mysqladmin processlist|grep -i locked|awk '{print $1}'`
do
   mysqladmin kill ${id}
done
```




mysql 批量删除sleep状态的线程脚本：
```
#It is used to kill processlist of mysql sleep

while :
do
  n=`mysqladmin processlist -u账号 -p密码|grep -i sleep |wc -l`
  date=`date +%Y%m%d\[%H:%M:%S]`
  echo $n
  if [ "$n" -gt 10 ]
  then
  for i in `mysqladmin processlist -u账号 -p密码|grep -i sleep |awk '{print $2}'`
  do
     mysqladmin -uadmin -pxxxx kill $i
  done
  echo "sleep is too many I killed it " >> /tmp/sleep.log
  echo "$date : $n" >> /tmp/sleep.log
  fi               
  sleep 1
done
```
3) 方案3：杀掉当前所有的/指定用户所有的MySQL连接
```
# 杀掉当前所有的MySQL连接
mysqladmin -uroot -p processlist|awk -F "|" '{print $2}'|xargs -n 1 mysqladmin -uroot -p kill
# 杀掉指定用户运行的连接，这里为ddtest
mysqladmin -uroot -p processlist|awk -F "|" '{if($3 == "ddtest")print $2}'|xargs -n 1 mysqladmin -uroot -p kill
```