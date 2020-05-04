---
layout: post
title: Docker搭建MySQL集群(PXC)
categories: Docker MySQL
tags: Docker MySQL
---

* content
{:toc}


### MySQL集群的目的

* 能满足处理高并发请求的高性能要求
* 能处理单节点故障，保证高可用性

### MySQL集群的常见方案
#### Replication

* 速度快，但仅能保证弱一致性，适用于保存价值不高的数据，比如日志、帖子、新闻等。
* 采用master-slave结构，在master写入会同步到slave，能从slave读出；但在slave写入无法同步到master。
* 采用异步复制，master写入成功就向客户端返回成功，但是同步slave可能失败，会造成无法从slave读出的结果。

#### PXC (Percona XtraDB Cluster)

* 速度慢，但能保证强一致性，适用于保存价值较高的数据，比如订单、客户、支付等。
* 数据同步是双向的，在任一节点写入数据，都会同步到其他所有节点，在任何节点上都能同时读写。
* 采用同步复制，向任一节点写入数据，只有所有节点都同步成功后，才会向客户端返回成功。事务在所有节点要么同时提交，要么不提交。

### MySQL集群搭建

在Docker中安装PXC集群，使用Docker仓库中的PXC官方镜像

#### 从docker官方仓库中拉下PXC镜像

```shell
[root@docker01 ~]# docker pull percona/percona-xtradb-cluster
Using default tag: latest
Trying to pull repository docker.io/percona/percona-xtradb-cluster ... 
latest: Pulling from docker.io/percona/percona-xtradb-cluster
ff144d3c0ab1: Pull complete 
eafdff1524b5: Pull complete 
c281665399a2: Pull complete 
c27d896755b2: Pull complete 
c43c51f1cccf: Pull complete 
6eb96f41c54d: Pull complete 
4966940ec632: Pull complete 
2bafadcea292: Pull complete 
3c2c0e21b695: Pull complete 
52a8c2e9228e: Pull complete 
f3f28eb1ce04: Pull complete 
d301ece75f56: Pull complete 
3d24904bec3c: Pull complete 
1053c2982c37: Pull complete 
Digest: sha256:78460483e99c093d2910d3667d928ed8c2165165554482058875bccafa4ccf0b
Status: Downloaded newer image for docker.io/percona/percona-xtradb-cluster:latest
```

#### 重命名镜像

```shell
[root@docker01 ~]# docker tag percona/percona-xtradb-cluster:latest pxc
```

#### 创建网段
出于安全考虑，给PXC集群创建Docker内部网络

```shell
[root@docker01 ~]# docker network create --subnet=172.18.0.0/24 net1
13dd609ce49335a614e6c20a5a7a4b4d36b0d27cdbff5a55540ef10486777a2a
```


#### 创建Docker卷

> 使用Docker时，业务数据应保存在宿主机中，采用目录映射，这样可以使数据与容器独立。但是容器中的PXC无法直接使用映射目录，解决办法是采用Docker卷来映射

```shell
[root@docker01 ~]# docker volume create --name v1
v1
[root@docker01 ~]# docker volume create --name v2
v2
[root@docker01 ~]# docker volume create --name v3
v3
[root@docker01 ~]# docker volume create --name v4
v4
[root@docker01 ~]# docker volume create --name v5
v5
```

#### 创建PXC容器

```shell
# 创建5个PXC容器构成集群 
# 第一个节点 
$ docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql --name=node1 --network=net1 --ip 172.18.0.2 pxc 
# 在第一个节点启动后要等待一段时间，等候mysql启动完成。 
# 第二个节点 
$ docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v2:/var/lib/mysql --name=node2 --net=net1 --ip 172.18.0.3 pxc 
# 第三个节点 
$ docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v3:/var/lib/mysql --name=node3 --net=net1 --ip 172.18.0.4 pxc 
# 第四个节点 
$ docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v4:/var/lib/mysql --name=node4 --net=net1 --ip 172.18.0.5 pxc 
# 第五个节点 
$ docker run -d -p 3311:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v5:/var/lib/mysql --name=node5 --net=net1 --ip 172.18.0.6 pxc 
```
> 我按照教程的命令启动PXC容器时一直失败，报如下错误：
> mkdir: cannot create directory ‘’: No such file or directory
> 后来查找了半天，发现命令里面不能带 "–privileged"参数，去掉这个参数后节点启动正常

> 另外命令参数中的密码设置我一开始是"root"，结果会造成PXC容器启动一段时间就自动退出，或者SQL客户端无法连接。
> 后来我把密码改成"abc123456"，就好了。很玄妙，我暂时还不知道为什么，但感觉密码最好字母加数字，不要是"root" 




### 数据库集群负载均衡

将请求均匀地发送给集群中的每一个节点。

* 所有请求发送给单一节点，其负载过高，性能很低，而其他节点却很空闲。
* 用Haproxy做负载均衡，可以将请求均匀地发送给每个节点，单节点负载低，性能好

![Haproxy-PXC](https://www.aiops.work/images/mysql/mysql01.png)

### 负载均衡中间件对比

![Haproxy-PXC](https://www.aiops.work/images/mysql/mysql03.png)

### Haproxy安装

#### 从Docker仓库拉取haproxy镜像

```shell
[root@docker01 ~]# docker pull haproxy
Using default tag: latest
Trying to pull repository docker.io/library/haproxy ... 
latest: Pulling from docker.io/library/haproxy
743f2d6c1f65: Pull complete 
8189ac8398db: Pull complete 
8c16b00b1545: Pull complete 
Digest: sha256:4538ea1c4309821daa8143579e697f3a6c06dad9dc09fc750d67698469ea37a0
Status: Downloaded newer image for docker.io/haproxy:latest
[root@docker01 ~]# docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
docker.io/haproxy                          latest              0b132e159de1        9 days ago          72.2 MB
docker.io/percona/percona-xtradb-cluster   latest              70b3670450ef        3 months ago        408 MB
pxc                                        latest              70b3670450ef        3 months ago        408 MB
docker.io/tomcat                           8.5.32              5808f01b11bf        9 months ago        463 MB

```



#### 创建Haproxy配置文件。供Haproxy容器使用

```shell
[root@docker01 ~]# mkdir /home/soft/haproxy/ -p
[root@docker01 ~]# mkdir /usr/local/etc/haproxy -p
[root@docker01 ~]# vim /home/soft/haproxy/haproxy.cfg
#haproxy.conf写入下列内容，具体配置需要修改
global
    #工作目录
    chroot /usr/local/etc/haproxy
    #日志文件，使用rsyslog服务中local5日志设备（/var/log/local5），等级info
    log 127.0.0.1 local5 info
    #守护进程运行
    daemon

defaults
    log global
    mode    http
    #日志格式
    option  httplog
    #日志中不记录负载均衡的心跳检测记录
    option  dontlognull
    #连接超时（毫秒）
    timeout connect 5000
    #客户端超时（毫秒）
    timeout client  50000
    #服务器超时（毫秒）
    timeout server  50000

#监控界面
listen  admin_stats
    #监控界面的访问的IP和端口
    bind  0.0.0.0:8888
    #访问协议
    mode        http
    #URI相对地址
    stats uri   /dbs
    #统计报告格式
    stats realm     Global\ statistics
    #登陆帐户信息
    stats auth  admin:admin123456
#数据库负载均衡
listen  proxy-mysql
    #访问的IP和端口
    bind  0.0.0.0:3306
    #网络协议
    mode  tcp
    #负载均衡算法（轮询算法）
    #轮询算法：roundrobin
    #权重算法：static-rr
    #最少连接算法：leastconn
    #请求源IP算法：source
    balance  roundrobin
    #日志格式
    option  tcplog
    #在MySQL中创建一个没有权限的haproxy用户，密码为空。Haproxy使用这个账户对MySQL数据库心跳检测
    option  mysql-check user haproxy
    server  MySQL_1 172.18.0.2:3306 check weight 1 maxconn 2000
    server  MySQL_2 172.18.0.3:3306 check weight 1 maxconn 2000
    server  MySQL_3 172.18.0.4:3306 check weight 1 maxconn 2000
    server  MySQL_4 172.18.0.5:3306 check weight 1 maxconn 2000
    server  MySQL_5 172.18.0.6:3306 check weight 1 maxconn 2000
    #使用keepalive检测死链
    option  tcpka
```

#### 创建mysql的haproxy用户

> 进入pxc某个节点，在mysql中创建haproxy用户。haproxy中间件需要使用该账号登陆数据库，发送心跳检测。

```shell
[root@docker01 ~]# docker exec -it node2 mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.7.25-28-57 Percona XtraDB Cluster (GPL), Release rel28, Revision a2ef85f, WSREP version 31.35, wsrep_31.35

Copyright (c) 2009-2019 Percona LLC and/or its affiliates
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> create user 'haproxy'@'%' identified by '';
Query OK, 0 rows affected (0.03 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

```

#### 创建Haproxy容器
```shell
[root@docker01 ~]# docker run -it -d -p 4001:8888 -p 4002:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h1 --net=net1 --ip 172.18.0.7 --privileged haproxy
// -p 端口号 8888 ：haproxy后台监控界面端口，与haproxy.cnf中的配置一致
// -p 端口号 3306 ：haproxy对外提供负载均衡服务端口，可以直接通过该端口连接到pxc集群
// -v 目录映射，把宿主机的配置文件所在目录映射到容器的工作目录中
// --name 容器名，建议带数字编号1，高可用时haproxy需要配置成双节点
// --net 网段需要和pxc集群在同一网段中
// --ip 可以指定IP，当不指定时docker虚拟机会默认分配一个
```


#### 进入容器执行命令启动Haproxy	

> [root@docker01 haproxy]# docker exec -it h1 bash



#### 在容器bash中启动Haproxy

> root@e3a9de34f26d:/# haproxy -f /usr/local/etc/haproxy/haproxy.cfg

#### 验证haproxy

* 通过浏览器访问haproxy监控页面，远程访问需要开放端口

* 访问地址：http://宿主机IP:4001/dbs

* 访问路径配置文件中：stats uri /dbs

* 用户名、密码在配置文件中：stats auth admin:admin123456
* Haproxy不存储数据，只转发数据。可以在数据库中建立Haproxy的连接，端口4002，用户名和密码为数据库集群的用户名和密码

![Haproxy-PXC](https://www.aiops.work/images/mysql/mysql04.png)

### Haproxy冗余设计

> 单节点的Haproxy不具备高可用性，必须进行冗余设计，采用双节点或多节点。

### 虚拟IP地址

> linux系统可以在一个网卡中定义多个IP地址，把这些地址分配给多个应用程序，这些地址就是虚拟IP，Haproxy的双机热备方案最关键的技术就是虚拟IP。

![Haproxy-PXC](https://www.aiops.work/images/mysql/mysql05.png)

* 定义虚拟IP
* 在Docker中启动两个Haproxy容器，每个容器中还需要安装Keepalived程序（以下简称KA）
* 两个KA会争抢虚拟IP，一个抢到后，另一个没抢到就会等待，抢到的作为主服务器，没抢到的作为备用服务器
* 两个KA之间会进行心跳检测，如果备用服务器没有受到主服务器的心跳响应，说明主服务器发生故障，那么备用服务器就可以争抢虚拟IP，继续工作
* 我们向虚拟IP发送数据库请求，一个Haproxy挂掉，可以有另一个接替工作

### Haproxy双机热备方案

![Haproxy-PXC](https://www.aiops.work/images/mysql/mysql02.png)

* Docker中创建两个Haproxy，并通过Keepalived抢占Docker内地虚拟IP
* Docker内的虚拟IP不能被外网，所以需要借助宿主机Keepalived映射成外网可以访问地虚拟IP



#### Docker配置Keepalived
```shell
[root@docker01 ~]# docker exec -it h1 bash
root@e3a9de34f26d:/# apt-get update
Get:1 http://security-cdn.debian.org/debian-security stretch/updates InRelease [94.3 kB]              
Ign:2 http://cdn-fastly.deb.debian.org/debian stretch InRelease                                       
Get:3 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]
Get:4 http://cdn-fastly.deb.debian.org/debian stretch Release [118 kB]                      
Get:5 http://cdn-fastly.deb.debian.org/debian stretch-updates/main amd64 Packages [27.2 kB] 
Get:6 http://cdn-fastly.deb.debian.org/debian stretch Release.gpg [2434 B]       
Get:7 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 Packages [7082 kB]                            
Get:8 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 Packages [492 kB]              
Fetched 7907 kB in 1min 40s (78.7 kB/s)                                                                        
Reading package lists... Done
root@e3a9de34f26d:/# apt-get install keepalived -y


```



```shell

state MASTER # Keepalived的身份（MASTER主服务要抢占IP，BACKUP备服务器不会抢占IP） 
interface eth0 #docker网卡设备，虚拟IP所在 
virtual_router_id 51 #虚拟路由标识，MASTER和BACKUP的虚拟路由标识必须一致。从0～255 
priority 100 # MASTER权重要高于BACKUP数字越大优先级越高 
advert_int 1 # MASTER和BACKUP节点同步检查的时间间隔，单位为秒，主备之间必须一致 
authentication# 主从服务器验证方式。主备必须使用相同的密码才能正常通信 
virtual_ipaddress # 虚拟IP。可以设置多个虚拟IP地址，每行一个 

[root@docker01 ~]# vim keepalived.conf
vrrp_instance VI_1 {
    state  MASTER
    interface  eth0
    virtual_router_id  51
    priority  100
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  123456
    }
     virtual_ipaddress {
        172.18.0.201
    }
}


```

```shell
[root@docker01 ~]# docker cp keepalived.conf h1:/etc/keepalived/
[root@docker01 ~]# docker exec -it h1 bash
root@e3a9de34f26d:/# service keepalived start
[ ok ] Starting keepalived: keepalived.
root@e3a9de34f26d:/# ping 172.18.0.201
PING 172.18.0.201 (172.18.0.201) 56(84) bytes of data.
64 bytes from 172.18.0.201: icmp_seq=1 ttl=64 time=0.094 ms
64 bytes from 172.18.0.201: icmp_seq=2 ttl=64 time=0.121 ms
64 bytes from 172.18.0.201: icmp_seq=3 ttl=64 time=0.098 ms
64 bytes from 172.18.0.201: icmp_seq=4 ttl=64 time=0.098 ms

root@e3a9de34f26d:/# apt-get install net-tools
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  net-tools
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 248 kB of archives.
After this operation, 963 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stretch/main amd64 net-tools amd64 1.60+git20161116.90da8a0-1 [248 kB]
Fetched 248 kB in 6s (37.1 kB/s)                                                                                                                                                                                
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package net-tools.
(Reading database ... 10555 files and directories currently installed.)
Preparing to unpack .../net-tools_1.60+git20161116.90da8a0-1_amd64.deb ...
Unpacking net-tools (1.60+git20161116.90da8a0-1) ...
Setting up net-tools (1.60+git20161116.90da8a0-1) ...
root@e3a9de34f26d:/# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      8/haproxy           
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      8/haproxy           
tcp        0      0 127.0.0.11:44688        0.0.0.0:*               LISTEN      -  
```

#### Haproxy 2
```shell
[root@docker01 ~]# docker run -it -d -p 4003:8888 -p 4004:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h2 --net=net1 --ip 172.18.0.8 --privileged haproxy
43d5427e93b99f09ba474ff99685163c1ad64a077c0735624198f08405194c4f
[root@docker01 ~]# docker exec -it h2 bash
root@43d5427e93b9:/# haproxy -f /usr/local/etc/haproxy/haproxy.cfg
root@43d5427e93b9:/# apt-get update
Ign:2 http://cdn-fastly.deb.debian.org/debian stretch InRelease                                                
Get:3 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]                              
Get:1 http://security-cdn.debian.org/debian-security stretch/updates InRelease [94.3 kB]
Get:4 http://cdn-fastly.deb.debian.org/debian stretch Release [118 kB]               
Get:5 http://cdn-fastly.deb.debian.org/debian stretch-updates/main amd64 Packages [27.2 kB]
Get:6 http://cdn-fastly.deb.debian.org/debian stretch Release.gpg [2434 B]       
Get:7 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 Packages [7082 kB]
Get:8 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 Packages [492 kB]
Fetched 7907 kB in 2min 26s (54.1 kB/s)                                                                        
Reading package lists... Done
root@43d5427e93b9:/# apt-get install keepalived -y


[root@docker01 ~]# cp keepalived.conf keepalived.conf-h2
[root@docker01 ~]# vim keepalived.conf-h2

vrrp_instance VI_1 {
    state  BACKUP
    interface  eth0
    virtual_router_id  51
    priority  80
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  123456
    }
     virtual_ipaddress {
        172.18.0.201
    }
}

[root@docker01 ~]# docker cp keepalived.conf-h2 h2:/etc/keepalived/keepalived.conf



root@43d5427e93b9:/# cd /etc/keepalived/
root@43d5427e93b9:/etc/keepalived# more keepalived.conf 
vrrp_instance VI_1 {
    state  BACKUP
    interface  eth0
    virtual_router_id  51
    priority  80
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  123456
    }
     virtual_ipaddress {
        172.18.0.201
    } 
}
root@43d5427e93b9:/etc/keepalived# service keepalived start
[ ok ] Starting keepalived: keepalived.
root@43d5427e93b9:/# ping 172.18.0.201
PING 172.18.0.201 (172.18.0.201) 56(84) bytes of data.
64 bytes from 172.18.0.201: icmp_seq=1 ttl=64 time=0.268 ms
64 bytes from 172.18.0.201: icmp_seq=2 ttl=64 time=0.156 ms

```








### 实现外网访问虚拟IP

```shell
[root@docker01 ~]# yum install -y openssl openssl-devel keepalived
[root@docker01 ~]# vim /etc/keepalived/keepalived.conf

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    } 
	virtual_ipaddress {
        192.168.8.191
#这里是指定的一个宿主机上的虚拟ip，一定要和宿主机网卡在同一个网段，
    } 
} 

#接受监听数据来源的端口，网页入口使用
virtual_server 192.168.8.191 8888 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
#把接受到的数据转发给docker服务的网段及端口，由于是发给docker服务，所以和docker服务数据要一致
    real_server 172.18.0.201 8888 {
        weight 1
    } 
} 

#接受数据库数据端口，宿主机数据库端口是3306，所以这里也要和宿主机数据接受端口一致
virtual_server 192.168.8.191 3306 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 172.18.0.201 3306 {
        weight 1
    } 
}


[root@docker01 ~]# systemctl start keepalived
[root@docker01 ~]# yum install net-tools -y
[root@docker01 ~]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      9411/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      9836/master         
tcp6       0      0 :::3307                 :::*                    LISTEN      70638/docker-proxy- 
tcp6       0      0 :::3308                 :::*                    LISTEN      70904/docker-proxy- 
tcp6       0      0 :::3309                 :::*                    LISTEN      72024/docker-proxy- 
tcp6       0      0 :::3310                 :::*                    LISTEN      73232/docker-proxy- 
tcp6       0      0 :::3311                 :::*                    LISTEN      74437/docker-proxy- 
tcp6       0      0 :::22                   :::*                    LISTEN      9411/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      9836/master         
tcp6       0      0 :::4001                 :::*                    LISTEN      77252/docker-proxy- 
tcp6       0      0 :::4002                 :::*                    LISTEN      77265/docker-proxy- 
tcp6       0      0 :::4003                 :::*                    LISTEN      80591/docker-proxy- 
tcp6       0      0 :::4004                 :::*                    LISTEN      80577/docker-proxy-

[root@docker01 ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:84:45:75 brd ff:ff:ff:ff:ff:ff
    inet 192.168.8.211/24 brd 192.168.8.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.8.191/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::c71:19f9:8df7:f491/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:08:94:f1:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:8ff:fe94:f103/64 scope link 
       valid_lft forever preferred_lft forever
12: br-13dd609ce493: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:b9:89:cb:b0 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/24 scope global br-13dd609ce493
       valid_lft forever preferred_lft forever
    inet6 fe80::42:b9ff:fe89:cbb0/64 scope link 
       valid_lft forever preferred_lft forever
14: veth5d9969d@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether 26:e0:af:e0:2e:a0 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::24e0:afff:fee0:2ea0/64 scope link 
       valid_lft forever preferred_lft forever
16: veth9bad7f6@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether aa:23:c6:af:c8:be brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::a823:c6ff:feaf:c8be/64 scope link 
       valid_lft forever preferred_lft forever
18: veth6a73b97@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether ca:f0:23:ef:96:dc brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::c8f0:23ff:feef:96dc/64 scope link 
       valid_lft forever preferred_lft forever
20: vetha3ca3d6@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether 9a:c5:7d:f9:0b:71 brd ff:ff:ff:ff:ff:ff link-netnsid 3
    inet6 fe80::98c5:7dff:fef9:b71/64 scope link 
       valid_lft forever preferred_lft forever
22: vethc1a1299@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether 4e:ac:2f:fa:11:1e brd ff:ff:ff:ff:ff:ff link-netnsid 4
    inet6 fe80::4cac:2fff:fefa:111e/64 scope link 
       valid_lft forever preferred_lft forever
38: vetha988561@if37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether 5a:d3:66:6c:a5:6b brd ff:ff:ff:ff:ff:ff link-netnsid 6
    inet6 fe80::58d3:66ff:fe6c:a56b/64 scope link 
       valid_lft forever preferred_lft forever
40: veth5057bf3@if39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-13dd609ce493 state UP group default 
    link/ether b6:36:84:f8:a4:31 brd ff:ff:ff:ff:ff:ff link-netnsid 7
    inet6 fe80::b436:84ff:fef8:a431/64 scope link 
       valid_lft forever preferred_lft forever

```









