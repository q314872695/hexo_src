---
title: mysql常用sql语句总结
date: 2019-08-29 15:24:00
tags: mysql
---



## 增

### 增加一条数据
```sql
insert into table_name ( field1, field2,...fieldN )
                       values
                       ( value1, value2,...valueN )
```
如果数据是字符型，必须使用单引号或者双引号，如："value"。

## 删
### 删除一条数据
```sql
delete from table_name [where Clause]
```
如果没有指定 WHERE 子句，MySQL 表中的所有记录将被删除。

## 改
### 更新一条数据
```sql
UPDATE table_name SET field1=new-value1, field2=new-value2
[WHERE Clause]
```

## 查
### 查询关键字的定义顺序
```sql
SELECT DISTINCT <select_list>
FROM <left_table>
<join_type> JOIN <right_table>
ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```
#### 用户操作

##### 创建用户

```sql
create user '用户名'@'主机名' identified by '密码';
```

- 主机名，例如`localhost`表示只能本机登录，`%`表示任意主机都能登录

##### 删除用户

```sql
drop user '用户名'@'主机名';
```

​	参数同上

##### 修改密码

```sql
set password for '用户名'@'主机名' = password('新密码');
set password = password('新密码'); //修改当前登录用户的密码
```

#### 权限操作

##### 授予权限

```sql
grant 权限名 on 数据库名.表名 to 'username'@'host'
```

- 权限名，如select，insert，update等，如果要授予所的权限则使用all



