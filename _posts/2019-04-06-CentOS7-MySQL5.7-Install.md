---
layout: post
title: CentOS7安装MySQL5.7
categories: MySQL
tags: MySQL CentOS7 
---

* content
{:toc}

### 安装前的检查

#### 检查Linux系统

```shell
[root@mysql5507 ~]# cat /etc/system-release
CentOS Linux release 7.6.1810 (Core)
[root@mysql5507 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819         132         153           9        1533        1468
Swap:          2047           0        2047
[root@mysql5507 ~]# cat /proc/cpuinfo| grep "processor"| wc -l
1


```

#### 检查是否安装了MySQL

```shell
[root@mysql5507 ~]# rpm -qa | grep mysql
[root@mysql5507 ~]# rpm -qa | grep mariadb
mariadb-libs-5.5.60-1.el7_5.x86_64
[root@mysql5507 ~]# systemctl stop mariadb
Failed to stop mariadb.service: Unit mariadb.service not loaded.
[root@mysql5507 ~]# rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
[root@mysql5507 ~]# rpm -qa | grep mariadb
```

#### 检查是否开启防火墙
```shell
[root@mysql5507 ~]# firewall-cmd --state
running
[root@mysql5507 ~]# systemctl stop firewalld.service
[root@mysql5507 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@mysql5507 ~]# systemctl enable firewalld.service
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
[root@mysql5507 ~]# systemctl start firewalld
[root@mysql5507 ~]# firewall-cmd --state
running
[root@mysql5507 ~]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
[root@mysql5507 ~]# firewall-cmd --reload
success
[root@mysql5507 ~]# firewall-cmd --zone=public --list-ports
3306/tcp
[root@mysql5507 ~]# systemctl stop firewalld
[root@mysql5507 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@mysql5507 ~]# firewall-cmd --state
not running
```

#### 检查NUMA

```shell
[root@mysql5507 ~]# rpm -qa | grep libaio
libaio-0.3.109-13.el7.x86_64
[root@mysql5507 ~]# numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0
node 0 size: 2047 MB
node 0 free: 718 MB
node distances:
node   0 
  0:  10 
```


#### 检查ulimit
```shell 
[root@mysql5507 ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7183
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7183
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

#### 检查依赖包libaio
```shell
[root@mysql5507 ~]# yum install -y libaio
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.cn99.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Package libaio-0.3.109-13.el7.x86_64 already installed and latest version
Nothing to do
```

#### 检查selinux
```shell
[root@mysql5507 ~]# getenforce
Enforcing
[root@mysql5507 ~]# sed -i "s#SELINUX=enforcing#SELINUX=disabled#g" /etc/sysconfig/selinux
[root@mysql5507 ~]# more /etc/sysconfig/selinux 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 


[root@mysql5507 ~]# getenforce
Enforcing
[root@mysql5507 ~]# setenforce 0
[root@mysql5507 ~]# getenforce
Permissive
```



### 安装MySQL

#### 创建用户和目录

```shell
[root@mysql5507 ~]# groupadd mysql
[root@mysql5507 ~]# useradd -g mysql -d /usr/local/mysql -s /sbin/nologin -M mysql
[root@mysql5507 ~]# id mysql
uid=1001(mysql) gid=1001(mysql) groups=1001(mysql)
[root@mysql5507 ~]# mkdir /opt/mysql
[root@mysql5507 ~]# tar zxf mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz -C /opt/mysql
[root@mysql5507 ~]# cd /usr/local
[root@mysql5507 local]# ln -s /opt/mysql/mysql-5.7.18-linux-glibc2.5-x86_64/  mysql
[root@mysql5507 local]# cd mysql/
[root@mysql5507 mysql]# pwd
/usr/local/mysql
[root@mysql5507 mysql]# cd ..
[root@mysql5507 local]# chown -R mysql.mysql mysql
[root@mysql5507 local]# ll
total 0
drwxr-xr-x. 2 root  root   6 Apr 11  2018 bin
drwxr-xr-x. 2 root  root   6 Apr 11  2018 etc
drwxr-xr-x. 2 root  root   6 Apr 11  2018 games
drwxr-xr-x. 2 root  root   6 Apr 11  2018 include
drwxr-xr-x. 2 root  root   6 Apr 11  2018 lib
drwxr-xr-x. 2 root  root   6 Apr 11  2018 lib64
drwxr-xr-x. 2 root  root   6 Apr 11  2018 libexec
lrwxrwxrwx. 1 mysql mysql 46 Apr  6 12:53 mysql -> /opt/mysql/mysql-5.7.18-linux-glibc2.5-x86_64/
drwxr-xr-x. 2 root  root   6 Apr 11  2018 sbin
drwxr-xr-x. 5 root  root  49 Apr  6 10:45 share
drwxr-xr-x. 2 root  root   6 Apr 11  2018 src
[root@mysql5507 local]# cd mysql/
[root@mysql5507 mysql]# ll
total 36
drwxr-xr-x.  2 root root   4096 Apr  6 12:53 bin
-rw-r--r--.  1 7161 31415 17987 Mar 18  2017 COPYING
drwxr-xr-x.  2 root root     55 Apr  6 12:53 docs
drwxr-xr-x.  3 root root   4096 Apr  6 12:52 include
drwxr-xr-x.  5 root root    229 Apr  6 12:53 lib
drwxr-xr-x.  4 root root     30 Apr  6 12:53 man
-rw-r--r--.  1 7161 31415  2478 Mar 18  2017 README
drwxr-xr-x. 28 root root   4096 Apr  6 12:53 share
drwxr-xr-x.  2 root root     90 Apr  6 12:53 support-files
```

#### 创建配置文件my.cnf
```shell
[root@mysql5507 ~]# vim /etc/my.cnf


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


```

#### 创建mysql的数据，日志，文件路径

```shell
[root@mysql5507 ~]# mkdir /data/mysql/mysql3306/{data,logs,tmp} -p
[root@mysql5507 ~]# chown -R mysql.mysql /data/mysql
[root@mysql5507 ~]# chmod -R 755 /data/mysql/
[root@mysql5507 mysql]# mysql
-bash: mysql: command not found
[root@mysql5507 mysql]# echo "export PATH=$PATH:/usr/local/mysql/bin" >>/etc/profile
[root@mysql5507 mysql]# more /etc/profile  | grep mysql
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/usr/local/mysql/bin
[root@mysql5507 mysql]# source /etc/profile
```

#### 初始化
```shell
[root@mysql5507 data]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize
2019-04-06T10:07:05.267433Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2019-04-06T10:07:05.281852Z 0 [Warning] InnoDB: Using innodb_file_format is deprecated and the parameter may be removed in future releases. See http://dev.mysql.com/doc/refman/5.7/en/innodb-file-format.html
 100 200
 100 200
2019-04-06T10:07:05.824590Z 0 [Warning] InnoDB: New log files created, LSN=45790
2019-04-06T10:07:05.852577Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2019-04-06T10:07:05.918667Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: bb4955f2-5853-11e9-9c58-000c29e108c0.
2019-04-06T10:07:05.920915Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2019-04-06T10:07:05.924518Z 1 [Note] A temporary password is generated for root@localhost: a:wKwwy7bCMJ

[root@mysql5507 mysql]# cp support-files/mysql.server /etc/init.d/mysql
[root@mysql5507 mysql]# /etc/init.d/mysql start
Starting MySQL.. SUCCESS! 


```


#### 登录
```shell
[root@mysql5507 data]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.18-log

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER USER() IDENTIFIED BY '123123';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on *.* to 'owen'@'%' identified by '123123' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```




