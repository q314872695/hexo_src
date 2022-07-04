---
title: mysql实现主从复制
tags: mysql
abbrlink: de3a676
date: 2022-06-10 15:29:58
---

# 前言
首先，你得准备两台服务器并且都安装了mysql，可以用虚拟机安装mysql在使用克隆功能克隆一个，需要注意的是克隆的那个mysql需要修改`uuid`，不然无法配置成功。或者也可以使用docker实现。
# 使用虚拟机实现
## 主库master配置
1. 修改mysql数据库的配置文件`/etc/mysql/my.cnf`，添加以下内容

> 题主使用的mysql5.7，其他版本配置文件位置可能稍有不同

```bash
[mysqld]
log-bin=mysql-bin # 启用二进制日志
server-id=100 # 服务器唯一ID，随便取一个只要主从数据库不一样就可以
```
修改完记得重启mysql

2. 登录数据库，执行下面SQL语句。

```sql
GRANT REPLICATION SLAVE ON *.* to '用户名'@'%' identified by '密码';
# 创建一个用户并授予权限（偷懒不创建新用户直接使用root用户也行）
```
3. 查看主库状态，需要记录`mysql-bin.000003`和`154`，此操作完成后不要才执行任何操作了，去配置从库吧，不然刚才记录的数值会变。
```bash
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

## 从库slave配置
1. 修改mysql数据库的配置文件`/etc/mysql/my.cnf`，添加一下内容
```bash
[mysqld]
server-id=101 # 服务器唯一ID
```
修改完记得重启mysql

 2. 登录mysql，分别执行一下SQL语句
```sql
change master to master_host='192.168.0.169' ,master_user= 'slave' , master_password='123456',master_log_file= 'mysql-bin.000002' ,master_log_pos=156;

start slave;
```
- 192.168.0.169为主库的ip
- slave和123456为刚才主库创建的用户名和密码
- mysql-bin.000003和154是刚才主库查询记录的值
3. 登录mysql执行`show slave status\G;`，查看从库状态
```sql
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: mysql_master
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 154
               Relay_Log_File: e4bd31e9ecec-relay-bin.000005
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 747
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 6c20332c-e734-11ec-9616-0242ac110002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```
这两个都是Yes时，则配置成功。
```bash
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```
# 使用docker实现
1. 创建一个网络
```bash
docker network create mysql-net
```
2. 创建主库
```bash
docker run -id --name mysql_master \
	-p 3306:3306 \
	-e MYSQL_ROOT_PASSWORD=123456 \
	--network mysql-net \
	mysql:5.7 \
	--server-id=1 \
	--log-bin=mysql-bin
```
创建mysql容器时可以直接指定配置，故不用修改配置文件了

3. 创建从库
```bash
docker run -id --name mysql_slave \
	-p 3307:3306 \
	-e MYSQL_ROOT_PASSWORD=123456 \
	--network mysql-net \
	mysql:5.7 \
	--server-id=2
```
4. 后面的步骤就一样了
