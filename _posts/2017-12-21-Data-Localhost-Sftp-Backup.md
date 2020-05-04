---
layout: post
title: 数据双重备份：本机备份+sftp备份
categories: 备份
tags: 备份 sftp
---

* content
{:toc}

### 备份方式
双重备份。即网站服务器本机备份和本地专用存储服务器备份。

### 传输方式
数据文件以sftp方式进行传输
#### 优点：
更安全。加密传输认证信息和传输的数据
#### 缺点：
效率低。使用了加密/解密技术，传输效率比普通的FTP要低得多。

### 自动备份思路


服务器|网站服务器 | 备份数据服务器
---|---|---
**IP**|192.168.33.191 | 192.168.33.230
**主机名**|robotweb|databak
**方法**|通过shell脚本自动打包压缩并备份 | 通过本机shell脚本执行sftp下载


### 基本安装与配置
以192.168.33.191模拟网站服务器，192.168.33.230模拟本地专用备份数据服务器。

#### sftp安装
省略，一般系统自带

#### 网站服务器操作


##### 创建备份脚本
```shell
[root@robotweb ~]# cd /usr/local/sh
[root@robotweb sh]# vim Auto_backup.sh

#!/bin/sh

MYDATE=`date +%F`
BACKUP=/datadatadata_app/backup/logs_bak

#压缩备份
zip -rq $BACKUP/ask_$MYDATE.zip /datadatadata_app/jetty-robot/webapps/robot/WEB-INF/logs/
zip -rq $BACKUP/robot_$MYDATE.zip /datadatadata_app/jetty-robot/logs/

#删除本地30天前的数据
rm -f $BACKUP/ask_$(date -d -30day +%F).zip
rm -f $BACKUP/robot_$(date -d -30day +%F).zip
```



##### 测验备份脚本
```python
[root@robotweb sh]# chmod +x Auto_backup.sh
[root@robotweb ~]# cd /datadatadata_app/backup/logs_bak/
[root@robotweb logs_bak]# ll
total 16
-rw-r--r--. 1 root root 5003 Dec 21 03:24 ask_2017-11-21.zip
-rw-r--r--. 1 root root 5834 Dec 21 03:24 robot_2017-11-21.zip

[root@robotweb logs_bak]# /usr/local/sh/Auto_backup.sh 
[root@robotweb logs_bak]# ll
total 16
-rw-r--r--. 1 root root 5003 Dec 21 03:25 ask_2017-12-21.zip
-rw-r--r--. 1 root root 6386 Dec 21 03:25 robot_2017-12-21.zip

[root@robotweb logs_bak]# mv ask_2017-12-21.zip ask_2017-11-21.zip 
[root@robotweb logs_bak]# mv robot_2017-12-21.zip robot_2017-11-21.zip 
[root@robotweb logs_bak]# ll
total 16
-rw-r--r--. 1 root root 5003 Dec 21 03:25 ask_2017-11-21.zip
-rw-r--r--. 1 root root 6386 Dec 21 03:25 robot_2017-11-21.zip
```

##### 添加计划任务
```shell
[root@robotweb logs_bak]# crontab -e

*       3       *       *       *       /usr/local/sh/Auto_backup.sh

[root@robotweb logs_bak]# crontab -l
*	3	*	*	*	/usr/local/sh/Auto_backup.sh


[root@robotweb logs_bak]# date -s 02:59:50
Thu Dec 21 02:59:50 EST 2017
[root@robotweb logs_bak]# ll
total 16
-rw-r--r--. 1 root root 5003 Dec 21 03:00 ask_2017-12-21.zip
-rw-r--r--. 1 root root 6509 Dec 21 03:00 robot_2017-12-21.zip
```








#### 备份数据服务器操作

##### 创建私钥和公钥 
```python
[root@databak ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
75:32:db:09:dd:94:43:02:82:f5:bc:30:61:d4:ec:e7 root@databak
The key's randomart image is:
+--[ RSA 2048]----+
|       +*o....o. |
|      .. =o. +o  |
|        o.B o .. |
|         +.O..   |
|        S ooo    |
|            E    |
|                 |
|                 |
|                 |
+-----------------+
[root@databak ~]# cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA2PYADGIZ1donRyHyVwojV2HAGNnoo4Xk/QXXnwHaSscBqRlkGMX9AqzJ+nG+n43lUb1P0R+Gy20pRXxe5vm3HB2czzzW28UdFVVlP3t34M6HQU+EavaCdIh3XvBmyaozt4xrR9B/zdDTWjencBfeJHnWS+xVZC7zNJytndI+yU+kPjQStQNXVNRIEPwtxkCjUUwJbMPIUPgYBiYXpTq0VikhRDvH5xwUVc9hH5WVFutLZhBOqKBX+WDviNUJW4nU0R02x8t/i5RUWK/wXWlIkWEzliYlFcJkb8rMqX7qBXeSrZwICsNBbhd+tgQW56wR9rBAjSvRM0Bud8PXT4FkQw== root@databak
[root@databak ~]# ssh-copy-id -i .ssh/id_rsa.pub root@192.168.33.191
The authenticity of host '192.168.33.191 (192.168.33.191)' can't be established.
RSA key fingerprint is 62:b1:7d:83:c6:53:b2:c6:97:ee:5c:02:c2:0e:14:09.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.33.191' (RSA) to the list of known hosts.
root@192.168.33.191's password: 
Now try logging into the machine, with "ssh 'root@192.168.33.191'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@databak ~]# sftp 192.168.33.191
Connecting to 192.168.33.191...
sftp>
```





```shell
[root@robotweb ~]# cd .ssh/
[root@robotweb .ssh]# ll
total 4
-rw-------. 1 root root 398 Dec 21 02:54 authorized_keys
```


##### 创建下载脚本
```shell
[root@databak ~]# cd /usr/local/sh
[root@databak sh]# vim Auto_sftp.sh

#!/bin/sh

MYDATE=`date +%F`
BACKUP=/datadatadata_app/backup/logs_bak

sftp 192.168.33.191 << EOM
get $BACKUP/ask_$MYDATE.zip /home/logs_bak
get $BACKUP/robot_$MYDATE.zip /home/logs_bak
bye
EOM
```


##### 验证下载脚本
```python
[root@databak sh]# chmod +x Auto_sftp.sh
[root@databak sh]# cd /home/logs_bak/
[root@databak logs_bak]# ll
总用量 0
[root@databak logs_bak]# /usr/local/sh/Auto_sftp.sh 
Connecting to 192.168.33.191...
sftp> get /datadatadata_app/backup/logs_bak/ask_2017-12-21.zip /home/logs_bak
Fetching /datadatadata_app/backup/logs_bak/ask_2017-12-21.zip to /home/logs_bak/ask_2017-12-21.zip
/datadatadata_app/backup/logs_bak/ask_2017-12-21.zip                                                   100% 5003     4.9KB/s   00:00    
sftp> get /datadatadata_app/backup/logs_bak/robot_2017-12-21.zip /home/logs_bak
Fetching /datadatadata_app/backup/logs_bak/robot_2017-12-21.zip to /home/logs_bak/robot_2017-12-21.zip
/datadatadata_app/backup/logs_bak/robot_2017-12-21.zip                                                 100% 6386     6.2KB/s   00:00    
sftp> bye
[root@databak logs_bak]# ll
总用量 16
-rw-r--r-- 1 root root 5003 12月 21 16:43 ask_2017-12-21.zip
-rw-r--r-- 1 root root 6386 12月 21 16:43 robot_2017-12-21.zip
```


##### 添加计划任务
```shell
[root@databak logs_bak]# crontab -e
0       5       *       *       *       /usr/local/sh/Auto_sftp.sh

[root@databak logs_bak]# crontab -l
0	5	*	*	*	/usr/local/sh/Auto_sftp.sh


[root@databak logs_bak]# rm * -fr
[root@databak logs_bak]# ll
总用量 0
[root@databak logs_bak]# date -s 04:59:00
2017年 12月 21日 星期四 04:59:00 CST
[root@databak logs_bak]# ll
总用量 16
-rw-r--r-- 1 root root 5003 12月 21 05:00 ask_2017-12-21.zip
-rw-r--r-- 1 root root 6386 12月 21 05:00 robot_2017-12-21.zip
[root@databak logs_bak]# date
2017年 12月 21日 星期四 05:00:10 CST
[root@databak logs_bak]# ntpdate 1.cn.pool.ntp.org
21 Dec 17:10:15 ntpdate[2909]: step time server 120.25.115.20 offset 43749.015826 sec
[root@databak logs_bak]# date
2017年 12月 21日 星期四 17:10:19 CST
```







