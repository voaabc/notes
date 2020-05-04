---
layout: post
title: MySQL参数及基本命令
categories: MySQL
tags: MySQL


---

* content
{:toc}

## 运行环境

### [Shell脚本部署MySQL5.6](https://www.datadatadata.cn/2017/12/23/Shell-install-MySQL5.6/)

### 添加参数



```java
[root@mysql01 ~]# vim /etc/my.cnf


[client]
port = 3306
socket = /data/mysql/data/mysql.sock

#添加客户端utf8
default-character-set = utf8


[mysqld]
user = mysql
port = 3306
server-id = 229
log-bin=mysql-bin

basedir = /usr/local/mysql
datadir = /data/mysql/data
socket = /data/mysql/data/mysql.sock

#添加如下参数

lower_case_table_names=1
max_allowed_packet=16M

innodb_strict_mode=1
innodb_buffer_pool_size=300M
innodb_stats_on_metadata=0
innodb_file_format=Barracuda
innodb_flush_method=O_DIRECT
innodb_log_files_in_group=2
innodb_log_file_size=256M
innodb_log_buffer_size=64M
innodb_file_per_table=1
innodb_max_dirty_pages_pct=60

key_buffer_size= 32M
tmp_table_size=32M
max_heap_table_size=32M
table_open_cache=1024
query_cache_type=0
query_cache_size=0
max_connections=1000
thread_cache_size=1024
open_files_limit=65535

default_storage_engine=InnoDB
character_set_server=utf8
collation_server=utf8_general_ci


---

[root@mysql01 ~]# service mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL......... SUCCESS! 
```

## 参数说明
百度谷歌

待补充


---


## 基本命令
### 密码修改

```java
[root@mysql01 ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.35 MySQL Community Server (GPL)

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
+--------------------+
3 rows in set (0.35 sec)

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
28 rows in set (0.00 sec)

mysql> update user set password = password('12345678') where user = 'root';
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye

[root@mysql01 ~]# mysql -uroot -p12345678 #生产不建议
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.35 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### 创建库

```java
mysql> show variables like '%char%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_database   | utf8                             |
| character_set_filesystem | binary                           |
| character_set_results    | utf8                             |
| character_set_server     | utf8                             |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysql/share/charsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.00 sec)

mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.33 sec)

mysql> create database datarobot;
Query OK, 1 row affected (0.35 sec)

mysql> create database datakbase;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| datakbase         |
| datarobot         |
+--------------------+
5 rows in set (0.00 sec)

```



---

### 创建用户


```java
mysql> create user 'data'@'%' identified by '123123';
Query OK, 0 rows affected (0.36 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> grant all privileges on *.* to 'root'@'%' identified by '12345678' with grant option;
Query OK, 0 rows affected (0.05 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select host,user from user;
+-----------+--------+
| host      | user   |
+-----------+--------+
| %         | mysync |
| %         | root   |
| %         | data  |
| 127.0.0.1 | root   |
| localhost | root   |
| mysql01   | root   |
+-----------+--------+
6 rows in set (0.00 sec)

```

### 授予权限


```java
mysql> grant all privileges on datarobot.* to data;  #生产不建议给所有权限
Query OK, 0 rows affected (0.35 sec)

mysql> grant all privileges on datakbase.* to data;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for data;
+------------------------------------------------------------------------------------------------------+
| Grants for data@%                                                                                   |
+------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'data'@'%' IDENTIFIED BY PASSWORD '*E56A114692FE0DE073F9A1DD68A00EEB9703F3F1' |
| GRANT ALL PRIVILEGES ON `datakbase`.* TO 'data'@'%'                                                |
| GRANT ALL PRIVILEGES ON `datarobot`.* TO 'data'@'%'                                                |
+------------------------------------------------------------------------------------------------------+
3 rows in set (0.00 sec)


```

