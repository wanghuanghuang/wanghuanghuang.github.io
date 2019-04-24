---
title: mysql 有则插入 无则更新
date: 2019-04-24 15:51:36
tags:
---
今天调查问题，遇到一个问题：一个表数据更新，有则插入，无则更新，因为插入与更新字段相同，所以可以直接使用replace into 没有问题，但是改了要求，表新增加一个字段，这一个字段在插入时需要赋值，更新时，这个字段不更新。replace into 不适用了。解决方案：使用 on duplicate key update.
1.replace into 
replace into 根据唯一索引或主键是否冲突决定插入或者更新。不冲突则插入 影响条数1条；冲突则先删除再插入，影响条数2条。不管传入参数是否与冲突参数相同。
```
REPLACE INTO alm_plan_vehicle (
	PLAN_ID,
	ALM_TYPE,
	CUSTOMER_ID,
	VEHICLE_ID,
	UPDATE_TIME,
	UPDATE_ID,
	CREATE_TIME
)
VALUES
	(
		381,
		9,
		234234,
		90862,
		'2019-04-1 09:44:49',
		46,
		'2019-04-11 09:44:49'
	)
```

2.on duplicate key update
on duplicate key update 根据主键或唯一索引是否冲突决定是插入还是更新。不冲突则插入，影响条数为1条；冲突则直接更新，影响条数2条，不会删除原记录，在其原记录的基础上更新；冲突时若传入参数与原参数一致，则不会去更新，影响条数为0条。
```
INSERT INTO alm_plan_vehicle (
	PLAN_ID,
	ALM_TYPE,
	CUSTOMER_ID,
	VEHICLE_ID,
	UPDATE_TIME,
	UPDATE_ID,
	CREATE_TIME
)
VALUES
	(
		381,
		9,
		234234,
		90862,
		'2019-04-1 09:44:49',
		46,
		'2019-04-11 09:44:49'
	) ON DUPLICATE KEY UPDATE 
	CUSTOMER_ID =
VALUES
	(CUSTOMER_ID),
	UPDATE_TIME =
VALUES
	(UPDATE_TIME),
	UPDATE_ID =
VALUES
	(UPDATE_ID)
```

上述sql含义：主键或者索引不冲突时，则是执行：
```
INSERT INTO alm_plan_vehicle (
	PLAN_ID,
	ALM_TYPE,
	CUSTOMER_ID,
	VEHICLE_ID,
	UPDATE_TIME,
	UPDATE_ID,
	CREATE_TIME
)
VALUES
	(
		381,
		9,
		234234,
		90862,
		'2019-04-1 09:44:49',
		46,
		'2019-04-11 09:44:49'
	)
```
主键或索引发生冲突时，则执行on duplicate key update 后的语句，如语句展示，只去更新CUSTOMER_ID、UPDATE_TIME、UPDATE_ID三个字段，且这三个值取得是传入进来的参数值。

