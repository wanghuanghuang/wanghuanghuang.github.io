---
title: inert-into-select语句使用
date: 2018-10-16 11:51:38
tags:
---


Insert into select 语句 顾名思义 就是 插入一查询出来的数据
举一个例子：
现有一个表：
```
sys_group_tag表：
CREATE TABLE `sys_group_tag` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `GROUP_ID` int(10) DEFAULT NULL,
  `TAG_ID` int(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx` (`GROUP_ID`,`TAG_ID`) USING HASH
) ENGINE=InnoDB AUTO_INCREMENT=65 DEFAULT CHARSET=utf8;
```
```
sys_tag表：
CREATE TABLE `sys_tag` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `tag_name` varchar(100) DEFAULT NULL COMMENT 'tag标签',
  `tag_type` tinyint(3) DEFAULT NULL COMMENT '标签类型  1第一受益人2催收 3 关注',
  `memo` varchar(254) DEFAULT NULL COMMENT '注释',
  `create_id` int(10) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_id` int(10) DEFAULT NULL COMMENT '更新人',
  `update_time` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `ISDEL` tinyint(3) DEFAULT '0' COMMENT '删除标记 0 可用 1 不可用',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8;
```
现有业务为：向sys_group_tag表中插入数据：group_id为1固定,tag_id是筛选sys_tag表中所有tag_type为1 并且未删除的id；
在navicat工具中 无法用表达式#{} 集合遍历直接写出,这时就用到insert into select 
实际语句为：
```
INSERT INTO 
dd_uni.sys_group_tag 
(group_id, tag_id)  
select '1',id 
from 
sys_tag 
where 
tag_type = 1 
AND isdel=0
```
不需要values，不需要括号 及时 两个select union查询 也不需要
还有一种是 若是在配置文件中如何编写？
现在讲解一种业务：还是上方两个表 但是 这次要插入的group_id不一定为1 是一个变化值，则实际sql为：
```
<insert id="saveUserTag">
INSERT INTO
   dd_uni.sys_group_tag
   (group_id,tag_id)
  SELECT
	#{group_id},
	id
  FROM
    sys_tag
  WHERE
    tag_type = 1
    AND isdel = 0
</insert> 
(#{group_id} 是传入的变化的值)
```

