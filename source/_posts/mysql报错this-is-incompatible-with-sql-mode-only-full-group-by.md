---
title: mysql报错this is incompatible with sql_mode=only_full_group_by
date: 2019-08-29 11:24:27
tags:
---

sql报错： Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'u.udcName' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

具体sql:
```mysql
SELECT
    u.udcName almName,
    u.udcCode almType,
    COUNT(p.id) planCount
FROM
    com_udc u
LEFT JOIN alm_plan p ON u.udcCode = p.ALM_TYPE
AND p.ISDEL = 0
AND p.PLAN_OWNER_TYPE = 1
AND p.customer_id IN
<foreach collection="customerIds" item="item" open="(" close=")" separator="," >
    #{item}
</foreach>
WHERE
    u.udcTypes = 'ALARM_TYPE'
GROUP BY
    almType
```

group by 用法：
select 选取的列 + 聚合函数 from 表名称 group by 分组列 ；执行时，先分组，在执行查询列或者where查询分组列。

ONLY_FULL_GROUP_BY意思：
对于group by操作，如果select中的列没有出现在group by中，那么这个sql是不合法的，如果查询列不在group by列中，但存在于select查询的聚合函数内也是可以的。

原因：
上述sql中，u.udcCode almType字段并不在聚合函数内也不存在与group by中。

解决方案：
先查询sql-mode设置：
    ```
    查看全局sql-mode设置：
    SELECT @@GLOBAL.sql_mode;
    查看会话sql-mode设置：
    SELECT @@SESSION.sql_mode;
    ```    
    
1）将select查询列加入到group by 条件后面，或者加入聚合函数中（不推荐）
2）设置sql-mode 去除默认ONLY_FULL_GROUP_BY： 
全局模式在线设置需要超级权限(SUPER)，新的连接才会生效；会话级别模式每个客户端都可设置。
    <1>命令行关闭ONLY_FULL_GROUP_BY
        ```
        （将查询的结果去除ONLY_FULL_GROUP_BY）
        set @@GLOBAL.sql_mode='';
            set sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
        ``` 
    
   这个时候 在用工具select 一下,发现已经不存在ONLY_FULL_GROUP_BY ，感觉已经OK。但是如果你重启Mysql服务的话，发现ONLY_FULL_GROUP_BY还是会存在的.
    <2>配置文件彻底解决：
    修改my.ini配置（mysql没有这个文件就把my-default.ini）
    在"mysqld"和"mysql"下添加：
    sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';（将查询的结果去除ONLY_FULL_GROUP_BY）

