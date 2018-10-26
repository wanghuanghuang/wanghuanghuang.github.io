---
title: Sql中and、or、union、union all 的用法
date: 2018-10-23 11:21:03
tags:
---


And 是“和、并且”的意思 
Or 是 “或者” 的意思
Union 与 union all 是 “联合” 的意思 区别为 union 去重 union all 不去重
举个例子：
	设计一张表 student
```
CREATE TABLE `student` (
  `id` int(10) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(10) DEFAULT NULL COMMENT '姓名',
  `sex` varchar(5) DEFAULT NULL COMMENT '性别',
  `age` int(5) DEFAULT NULL COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```
添加数据如图：
![数据图](/images/Sql中and_or_union_union_all 的用法/数据图.png)
1.	and 用法
挑选性别是男并且年龄为30岁的姓名
```
SQL：
   	Select s.name from student s where sex = '男' and age = 30
```
![and结果图](/images/Sql中and_or_union_union_all 的用法/and结果图.png)
2.	or 用法
挑选年龄为22 或 30 的人
```
SQL：
	Select s.name from student s where age = 22 or age = 30
```
![or结果图](/images/Sql中and_or_union_union_all 的用法/or结果图.png)
3.	union 用法 
4.	union all 用法
是联合查询 union 是A 条件的结果加上B条件的结果去重
Union all 是A 条件的结果加上B条件的结果 不去重 与or 相似


