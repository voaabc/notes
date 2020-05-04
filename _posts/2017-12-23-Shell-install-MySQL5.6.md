---
layout: post
title: Shell脚本部署MySQL5.6
categories: MySQL
tags: MySQL Shell
---

* content
{:toc}


### 安装前准备
```java
[root@mysql01 ~]# uname -a
Linux mysql01 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
[root@mysql01 ~]# cat /etc/issue
CentOS release 6.5 (Final)
Kernel \r on an \m

[root@mysql01 ~]# rpm -qa | grep mysql
mysql-server-5.1.73-8.el6_8.x86_64
mysql-libs-5.1.73-8.el6_8.x86_64
mysql-5.1.73-8.el6_8.x86_64
[root@mysql01 ~]# yum -y remove mysql*
[root@mysql01 ~]# rpm -qa | grep mysql
[root@mysql01 ~]# cd setup/
[root@mysql01 setup]# ll
total 307212
-rw-r--r--. 1 root root 314581668 Dec 17 08:56 mysql-5.6.35-linux-glibc2.5-x86_64.tar.gz
```


### 编写安装脚本


```python
[root@mysql01 setup]# vim MySQL5.6_install.sh
```

```shell
#! /bin/bash
source /etc/profile
# install MySQL5.6

#MySQL安装包
Mysql="mysql-5.6.35-linux-glibc2.5-x86_64"
#MySQL下载地址
Mysql_Url="http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.35-linux-glibc2.5-x86_64.tar.gz"
#MySQL安装目录
Mysql_Basedir="/usr/local"
#MySQL数据库目录
Mysql_data="/data/mysql"
Mysql_Datedir="$Mysql_data/data"
#MySQL用户
User=mysql
mkdir $Mysql_data/{tmp,binlog,log} -p


#网卡路径
Eth0='/etc/sysconfig/network-scripts/ifcfg-eth0' 
Em1='/etc/sysconfig/network-scripts/ifcfg-em1' 
if [ -f $Eth0 ];then 
IP_ADD="$Eth0"
else
if [ -f $Em1 ];then
IP_ADD=$Em1
else
echo "error The script cannot recognize the configuration file of the network card"
fi
fi
ip="`grep IPADDR $IP_ADD | awk -F '[ =]' '{print $2}'`"
#获取网卡的IP的最后一位
ip2="`grep IPADDR $IP_ADD | awk -F '[ =.]' '{print $5}'`"


echo "start install MySQL5.6"

#检查mysql用户是否存在，不存在则创建
egrep "^$User" /etc/passwd >& /dev/null  
if [ $? -ne 0 ];then  
    useradd -s /sbin/nologin $User  &> /dev/null
fi 
  
#安装libaio依赖包
echo "See Dependency installation package"
yum -y install  libaio* &> /dev/null
rpm -qa libaio*

# 下载MySQL5.6二进制包解压。
# wget $Mysql_Url 
#  如果本地没有MySQL的安装文件，可以取消前面的注释，会自动下载MySQL二进制安装包。
if [ -f $Mysql.tar.gz ];then
echo  "Unpack... $Mysql.tar.gz"
tar -xf $Mysql.tar.gz
else 
echo "$Mysql.tar.gz Package does not exist" 
fi


#移动mysql-5.6.35-linux-glibc2.5-x86_64 到$Mysql_Basedir/目录下改名为mysql
if [ -d $Mysql ];then
echo "move $Mysql mysql"
mv  $Mysql $Mysql_Basedir/mysql 
else 
echo "Unpack $Mysql.tar.gz ON" 
fi


#移动datadir 到$Mysql_Datedir
if [ -d $Mysql_Basedir/mysql/data ];then
echo "move $Mysql_Basedir/mysql/data  $Mysql_Datedir"
mv $Mysql_Basedir/mysql/data  $Mysql_Datedir
else 
echo "$Mysql_Basedir/mysql/data Non-existent" 
fi


#检查$Mysql_Datedir存在，给MySQL授权。

if [ -d $Mysql_Datedir ];then
chown -R mysql.mysql $Mysql_Datedir
chown -R mysql.mysql $Mysql_Basedir/mysql
else 
echo "$Mysql_Datedir Non-existent"
fi


#初始化MySQL
$Mysql_Basedir/mysql/scripts/mysql_install_db --datadir=$Mysql_Datedir --basedir=$Mysql_Basedir/mysql --user=$User --explicit_defaults_for_timestamp
if [ $? -eq 0 ];then  
echo "Initialization MySQL OK" 
else
echo "Initialization MySQL NO"
exit 4
fi


#拷贝启动脚本....
if [ ! -f /etc/init.d/mysqld ];then
cp $Mysql_Basedir/mysql/support-files/mysql.server /etc/init.d/mysqld
else 
rm -rf /etc/init.d/mysqld && cp $Mysql_Basedir/mysql/support-files/mysql.server /etc/init.d/mysqld
fi
echo "$Mysql_Basedir/mysql/lib" > /etc/ld.so.conf.d/mysql.conf  
if [ $? -eq 0 ];then 
echo "Mysql lib OK" 
fi
ln -s $Mysql_Basedir/mysql/include /usr/include/mysql
ln -s $Mysql_Basedir/mysql/bin/* /usr/bin/ 


#配置my.cnf
echo " " >/etc/my.cnf
echo "[client]
port = 3306
socket = $Mysql_Datedir/mysql.sock

[mysqld]
user = $User
port = 3306
server-id = $ip2
basedir = $Mysql_Basedir/mysql
datadir = $Mysql_Datedir
socket = $Mysql_Datedir/mysql.sock

default_storage_engine=InnoDB
character_set_server=utf8
collation_server=utf8_general_ci
" >/etc/my.cnf

#启动
service mysqld start

#删除默认用户配置
$Mysql_Basedir/mysql/bin/mysql -e "drop database test;drop user root@'::1'; DELETE FROM mysql.user WHERE user='';  select user,host from mysql.user;flush privileges;"
sleep 2

#初始化密码
/usr/local/mysql/bin/mysqladmin -uroot password '123123!'

#设置开机启动
chkconfig mysqld on

#重新启动
service mysqld restart
netstat -anput | grep mysqld 
Pid=`netstat  -anput | grep mysqld |  awk -F '[ ;]+' '{print $7}' |awk -F '[ /]+' '{print $1}'`
Port=`netstat  -anput | grep mysqld |  awk -F '[ ;]+' '{print $4}' | awk -F '[:;]+' '{print $2}'`
echo "PID  $Pid"  
echo "PORT $Port"

```

### 执行安装脚本
```java
[root@mysql01 setup]# chmod +x MySQL5.6_install.sh
[root@mysql01 setup]# ./MySQL5.6_install.sh 
start install MySQL5.6
See Dependency installation package
libaio-devel-0.3.107-10.el6.x86_64
libaio-0.3.107-10.el6.x86_64
Unpack... mysql-5.6.35-linux-glibc2.5-x86_64.tar.gz
move mysql-5.6.35-linux-glibc2.5-x86_64 mysql
move /usr/local/mysql/data  /data/mysql/data
WARNING: The host 'mysql01' could not be looked up with /usr/local/mysql/bin/resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !

Installing MySQL system tables...2017-12-23 09:14:15 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2017-12-23 09:14:15 0 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.6.35) starting as process 2002 ...
2017-12-23 09:14:16 2002 [Note] InnoDB: Using atomics to ref count buffer pool pages
2017-12-23 09:14:16 2002 [Note] InnoDB: The InnoDB memory heap is disabled
2017-12-23 09:14:16 2002 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2017-12-23 09:14:16 2002 [Note] InnoDB: Memory barrier is not used
2017-12-23 09:14:16 2002 [Note] InnoDB: Compressed tables use zlib 1.2.3
2017-12-23 09:14:16 2002 [Note] InnoDB: Using Linux native AIO
2017-12-23 09:14:16 2002 [Note] InnoDB: Using CPU crc32 instructions
2017-12-23 09:14:16 2002 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2017-12-23 09:14:16 2002 [Note] InnoDB: Completed initialization of buffer pool
2017-12-23 09:14:16 2002 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2017-12-23 09:14:16 2002 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2017-12-23 09:14:16 2002 [Note] InnoDB: Database physically writes the file full: wait...
2017-12-23 09:14:16 2002 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2017-12-23 09:14:17 2002 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2017-12-23 09:14:19 2002 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2017-12-23 09:14:19 2002 [Warning] InnoDB: New log files created, LSN=45781
2017-12-23 09:14:19 2002 [Note] InnoDB: Doublewrite buffer not found: creating new
2017-12-23 09:14:19 2002 [Note] InnoDB: Doublewrite buffer created
2017-12-23 09:14:19 2002 [Note] InnoDB: 128 rollback segment(s) are active.
2017-12-23 09:14:19 2002 [Warning] InnoDB: Creating foreign key constraint system tables.
2017-12-23 09:14:19 2002 [Note] InnoDB: Foreign key constraint system tables created
2017-12-23 09:14:19 2002 [Note] InnoDB: Creating tablespace and datafile system tables.
2017-12-23 09:14:19 2002 [Note] InnoDB: Tablespace and datafile system tables created.
2017-12-23 09:14:19 2002 [Note] InnoDB: Waiting for purge to start
2017-12-23 09:14:19 2002 [Note] InnoDB: 5.6.35 started; log sequence number 0
2017-12-23 09:14:20 2002 [Note] Binlog end
2017-12-23 09:14:20 2002 [Note] InnoDB: FTS optimize thread exiting.
2017-12-23 09:14:20 2002 [Note] InnoDB: Starting shutdown...
2017-12-23 09:14:22 2002 [Note] InnoDB: Shutdown completed; log sequence number 1625977
OK

Filling help tables...2017-12-23 09:14:22 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2017-12-23 09:14:22 0 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.6.35) starting as process 2024 ...
2017-12-23 09:14:22 2024 [Note] InnoDB: Using atomics to ref count buffer pool pages
2017-12-23 09:14:22 2024 [Note] InnoDB: The InnoDB memory heap is disabled
2017-12-23 09:14:22 2024 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2017-12-23 09:14:22 2024 [Note] InnoDB: Memory barrier is not used
2017-12-23 09:14:22 2024 [Note] InnoDB: Compressed tables use zlib 1.2.3
2017-12-23 09:14:22 2024 [Note] InnoDB: Using Linux native AIO
2017-12-23 09:14:22 2024 [Note] InnoDB: Using CPU crc32 instructions
2017-12-23 09:14:22 2024 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2017-12-23 09:14:22 2024 [Note] InnoDB: Completed initialization of buffer pool
2017-12-23 09:14:22 2024 [Note] InnoDB: Highest supported file format is Barracuda.
2017-12-23 09:14:22 2024 [Note] InnoDB: 128 rollback segment(s) are active.
2017-12-23 09:14:22 2024 [Note] InnoDB: Waiting for purge to start
2017-12-23 09:14:22 2024 [Note] InnoDB: 5.6.35 started; log sequence number 1625977
2017-12-23 09:14:22 2024 [Note] Binlog end
2017-12-23 09:14:22 2024 [Note] InnoDB: FTS optimize thread exiting.
2017-12-23 09:14:22 2024 [Note] InnoDB: Starting shutdown...
2017-12-23 09:14:24 2024 [Note] InnoDB: Shutdown completed; log sequence number 1625987
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  /usr/local/mysql/bin/mysqladmin -u root password 'new-password'
  /usr/local/mysql/bin/mysqladmin -u root -h mysql01 password 'new-password'

Alternatively you can run:

  /usr/local/mysql/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:

  cd . ; /usr/local/mysql/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl

  cd mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/

The latest information about MySQL is available on the web at

  http://www.mysql.com

Support MySQL by buying support/licenses at http://shop.mysql.com

New default config file was created as /usr/local/mysql/my.cnf and
will be used by default by the server when you start it.
You may edit this file to change server settings

Initialization MySQL OK
Mysql lib OK
Starting MySQL.Logging to '/data/mysql/data/mysql01.err'.
 SUCCESS! 
+------+-----------+
| user | host      |
+------+-----------+
| root | 127.0.0.1 |
| root | localhost |
| root | mysql01   |
+------+-----------+
Warning: Using a password on the command line interface can be insecure.
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
tcp        0      0 :::3306                     :::*                        LISTEN      2563/mysqld         
PID  2563
PORT 3306

```

### 查看安装状态
```python
[root@mysql01 setup]# ps aux | grep mysqld
root       2352  0.0  0.1  11432  1524 pts/2    S    09:14   0:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --daa --pid-file=/data/mysql/data/mysql01.pid
mysql      2563  0.5 45.0 1011676 451988 pts/2  Sl   09:14   0:00 /usr/local/mysql/bin/mysqld --basedir=/usr/lodata/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/data/mysql01.err l/data/mysql01.pid --socket=/data/mysql/data/mysql.sock --port=3306
root       2599  0.0  0.0 103244   856 pts/2    S+   09:15   0:00 grep mysqld
[root@mysql01 setup]# netstat -ntlp | grep mysqld
tcp        0      0 :::3306                     :::*                        LISTEN      2563/mysqld
[root@mysql01 setup]# reboot
[root@mysql01 ~]# netstat -ntlp | grep mysqld
tcp        0      0 :::3306                     :::*                        LISTEN      1214/mysqld
[root@mysql01 ~]# ps aux | grep mysqld
root       1003  0.0  0.1 108300  1620 ?        S    09:20   0:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql/data --pid-file=/data/mysql/data/mysql01.pid
mysql      1214  0.1 45.1 1077472 453648 ?      Sl   09:20   0:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/data/mysql01.err --pid-file=/data/mysql/data/mysql01.pid --socket=/data/mysql/data/mysql.sock --port=3306
root       1295  0.0  0.0 103244   856 pts/0    S+   09:24   0:00 grep mysqld
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
3 rows in set (0.06 sec)

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select host, user from user;
+-----------+------+
| host      | user |
+-----------+------+
| 127.0.0.1 | root |
| localhost | root |
| mysql01   | root |
+-----------+------+
3 rows in set (0.00 sec)

```




