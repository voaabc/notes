---
layout: post
title: MySQL5.6实现主从复制
categories: MySQL
tags: MySQL 主从复制
---

* content
{:toc}


## 原理
mysql要做到主从复制，其实依靠的是**二进制日志**，即：假设主服务器叫A，从服务器叫B；主从复制就是B跟着A学，A做什么，B就做什么。那么B怎么同步A的动作呢？现在A有一个日志功能，把自己所做的增删改查的动作全都记录在日志中，B只需要拿到这份日志，照着日志上面的动作施加到自己身上就可以了。这样就实现了主从复制。

  整体上来说，复制有3个步骤：   

1. master将改变记录到二进制日志(binary og)中（这些记录叫做二进制日志事件，binary log events）；
1. slave将master的binary log events拷贝到它的中继日志(relay log)；
1. slave重做中继日志中的事件，将改变反映它自己的数据。



## 环境

Hostnane | IP  | 系统  | 软件 | 规划
---|------|------|------|------
mysql01 | 192.168.1.229 | Centos 6.5|  MySQL5.6 |  master 
mysql02 | 192.168.1.230 | Centos 6.5|  MySQL5.6 |  slave  


## 安装
[Shell脚本部署MySQL5.6](http://www.bigdatadeeplearning.com/2017/12/23/Shell-install-MySQL5.6/)

## 配置

1. 主从服务器的 /etc/my.cnf 的配置，设置唯一ID 启用二进制日志。
1. 创建主从复制的账号，并授权REPLICATION SLAVE权限。
1. 查询master的状态，获取主服务器二进制日志信息。
1. 配置从服务器去连接主服务器进行数据复制。
1. 检查从服务器复制功能状态，测试主从复制。

### 修改主从服务器的配置文件

**修改主服务器master**

```java
[root@mysql01 ~]# vim /etc/my.cnf
[client]
port = 3306
socket = /data/mysql/data/mysql.sock

[mysqld]
user = mysql
port = 3306

#服务器唯一ID，必须整数
server-id = 229
#启用二进制日志，并设置二进制日志文件前缀
log-bin=mysql-bin


basedir = /usr/local/mysql
datadir = /data/mysql/data
socket = /data/mysql/data/mysql.sock

default_storage_engine=InnoDB
character_set_server=utf8
collation_server=utf8_general_ci
```
**注意**：在配置文件中不可以使用skip-networking参数选项，否则从服务器将无法与主服务器进行连接并复制数据。

**修改从服务器slave**


```java
[client]
port = 3306
socket = /data/mysql/data/mysql.sock

[mysqld]
user = mysql
port = 3306

server-id = 230
log-bin=mysql-bin

basedir = /usr/local/mysql
datadir = /data/mysql/data
socket = /data/mysql/data/mysql.sock

default_storage_engine=InnoDB
character_set_server=utf8
collation_server=utf8_general_ci


```
**注意**：如果有多台从服务器，则所有的服务器ID编号都必须是唯一的。MySQL从服务器上二进制日志功能是不需要开启的。但是，你也可以通过启用从服务器的二进制日志功能，实现数据备份与恢复，此外在一些更复杂的拓扑环境中，MySQL从服务器也可以扮演其他从服务器的主服务器。

**修改完成后，重启两台服务器的mysql**


```python
[root@mysql01 ~]# service mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 

[root@mysql02 ~]# service mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS!
```


### 主服务器上建立帐户并授权slave


```java
[root@mysql01 ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.35-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> GRANT REPLICATION SLAVE ON *.* to 'mysync'@'%' identified by '123456';
Query OK, 0 rows affected (0.06 sec)
```
这个账户必须拥有**REPLICATION SLAVE**权限，可以为不同的从服务器创建不同的账户与密码，也可以使用统一的账户与密码。


### 主服务器查询master的状态


```shell
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      320 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
File列显示的是二进制日志文件名，Position为当前日志记录位置，从服务器的设置中需要用到。

 **注意**：执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化防止其他主机操作主数据库，可以用只读锁表的方式来防止数据库被修改。
 

```java
mysql> flush tables with read lock;
Query OK, 0 rows affected (0.33 sec)

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      320 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```
 **flush tables with read lock;** 命令的作用是对所有数据库的所有表执行只读锁定，只读锁定后所有数据库的写操作将被拒绝，但读操作可以继续。执行锁定可以防止在查看二进制日志信息的同时有人对数据进行修改操作，最后使用**unlock tables;** 语句对全局锁执行结束操作。


**提示**：
如果MySQL数据库系统中已经存在大量数据，可以使用使用mysqldump工具在主服务器进行备份，然后导入从服务器。

**(主)导出**

```python
[root@mysql01 ~]# mysqldump -u root -p --all-databases --lock-all-tables > bak_mysql.sql
```

**(从)导入**

```python
[root@mysql02 ~]# mysqldump -u root -p --all-databases < bak_mysql.sql
```

### 配置从服务器Slave


数据复制的关键操作是配置从服务器去连接主服务器进行数据复制，我们需要告知从服务器建立网络连接所有必要的信息。

使用**CHANGE MASTER TO** 语句即可完成该项工作，

- **MASTER_HOST** 指定主服务器主机名或IP地址
- **MASTER_USER** 为主服务器上创建的拥有复制权限的账户名称
- **MASTER_PASSWORD** 为该账户的密码
- **MASTER_LOG_FILE** 指定主服务器二进制日志文件名称
- **MASTER_LOG_POS** 为主服务器二进制日志当前记录的位置


```java
[root@mysql02 ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.35-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> change master to master_host='192.168.1.229',master_user='mysync',master_password='123456',master_log_file='mysql-bin.000002',master_log_pos=320;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> start slave;   #启动从服务器复制功能
Query OK, 0 rows affected (0.33 sec)
```

### 检查从服务器复制功能状态


```java
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.229  //主服务器地址
                  Master_User: mysync  //授权帐户名，尽量避免使用root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 395    //同步读取二进制日志的位置，大于等于主服务器
               Relay_Log_File: mysql02-relay-bin.000003
                Relay_Log_Pos: 358
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes  //此状态必须YES
            Slave_SQL_Running: Yes  //此状态必须YES
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 395
              Relay_Log_Space: 533
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
             Master_Server_Id: 229
                  Master_UUID: 9f70b657-e2c7-11e7-9efc-000c29cb05c0
             Master_Info_File: /data/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

```

注意：**Slave_IO**及**Slave_SQL**进程必须正常运行，即**YES**状态，否则都是错误的状态(如：其中一个NO均属错误)。以上操作过程，主从服务器配置完成。

**关闭防火墙** chkconfig iptables off && service iptables stop

### 主从服务器测试

**主服务器Mysql，建立数据库，并在这个库中建表插入一条数据**：

```java
[root@mysql01 ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.6.35-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database test_db;
Query OK, 1 row affected (0.00 sec)

mysql> use test_db;
Database changed
mysql> create table test_db(id int(3),name char(10));
Query OK, 0 rows affected (0.11 sec)

mysql> insert into test_db values(001,'bobu');
Query OK, 1 row affected (0.04 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test_db            |
+--------------------+
4 rows in set (0.00 sec)
```

**从服务器Mysql查询**：


```java
[root@mysql02 ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.6.35-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test_db            |
+--------------------+
4 rows in set (0.00 sec)

mysql> use test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from test_db;     //查看主服务器上新增的具体数据
+------+------+
| id   | name |
+------+------+
|    1 | bobu |
+------+------+
1 row in set (0.00 sec)
```

MySQL主从复制完成。