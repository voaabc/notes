---
layout: post
title: Nginx+Keepalived实现双机热备的高可用
categories: Keepalived
tags: 高可用
---

* content
{:toc}


## 简介

Keepalived是一个免费开源的，用C编写的类似于layer3, 4 & 7交换机制软件，具备我们平时说的第3层、第4层和第7层交换机的功能。主要提供loadbalancing（负载均衡）和 high-availability（高可用）功能，负载均衡实现需要依赖Linux的虚拟服务内核模块（ipvs），而高可用是通过VRRP协议实现多台机器之间的故障转移服务。 

---
## 名词解释
### 双机热备
**双机热备**特指基于高可用系统中的两台服务器的热备（或高可用），因两机高可用在国内使用较多，故得名双机热备，双机高可用按工作中的切换方式分为：主-备方式（Active-Standby方式）和双主机方式（Active-Active方式），主-备方式即指的是一台服务器处于某种业务的激活状态（即Active状态），另一台服务器处于该业务的备用状态（即Standby状态)。而双主机方式即指两种不同业务分别在两台服务器上互为主备状态（即Active-Standby和Standby-Active状态）。

### 高可用性
**高可用性H.A.（High Availability**）指的是通过尽量缩短因日常维护操作（计划）和突发的系统崩溃（非计划）所导致的停机时间，以提高系统和应用的可用性。它与被认为是不间断操作的容错技术有所不同。HA系统是目前企业防止核心计算机系统因故障停机的最有效手段。


---

## 实验环境

Hostnane | IP  | 系统 | 应用 | VIP | 节点
---|------|------|------|------|----
web01 | 192.168.1.223 | Centos 6.5 |  Web server
web02 | 192.168.1.224 | Centos 6.5 |  Web server
nginx01 | 192.168.1.221 | Centos 6.5 |  Nginx+Keepalived | 192.168.1.220 |备节点
nginx02 | 192.168.1.222 | Centos 6.5 |  Nginx+Keepalived | 192.168.1.220 |主节点
 

## 安装与配置
### Nginx安装配置
[Nginx实现反向代理负载均衡与静态缓存](http://www.xiaoi.online/2017/12/18/Nginx-Reverse-Proxy-Load-Balance-Cache/)
### Keepalived安装


```shell
[root@nginx02 setup]# wget http://www.keepalived.org/software/keepalived-1.2.2.tar.gz
[root@nginx02 setup]# yum install popt-devel -y
[root@nginx02 setup]# tar xzf keepalived-1.2.2.tar.gz
[root@nginx02 setup]# cd keepalived-1.2.2
[root@nginx02 keepalived-1.2.2]# ./configure --prefix=/usr/local/keepalived
[root@nginx02 keepalived-1.2.2]# make && make install
```

###  将 keepalived 安装成 Linux 系统服务

```shell

[root@nginx02 ~]# mkdir /etc/keepalived
[root@nginx02 ~]# cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
[root@nginx02 ~]# cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/  
[root@nginx02 ~]# cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
[root@nginx02 ~]# ln -s /usr/local/sbin/keepalived /usr/sbin/
[root@nginx02 ~]# ln -s /usr/local/keepalived/sbin/keepalived /sbin/
[root@nginx02 ~]# chkconfig keepalived on


```
备节点安装同上

### 修改 Keepalived 配置文件


#### 主节点
```c++


[root@nginx02 ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
  ##keepalived自带的邮件提醒需要开启sendmail服务。建议用独立的监控或第三方SMTP
  ##标识本节点的字条串，通常为 hostname
  router_id 192.168.1.222
}
##keepalived会定时执行脚本并对脚本执行的结果进行分析，动态调整vrrp_instance
   的优先级。如果脚本执行结果为0，并且weight配置的值大于0，则优先级相应的增加。
   如果脚本执行结果非0，并且weight配置的值小于 0，则优先级相应的减少。其他情况，
   维持原本配置的优先级，即配置文件中priority对应的值。
vrrp_script chk_nginx {
   script "/etc/keepalived/nginx_check.sh" ## 检测 nginx 状态的脚本路径
   interval 2 ## 检测时间间隔
   weight -20 ## 如果条件成立，权重-20
}
## 定义虚拟路由，VI_1为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
   state MASTER ## 主节点为MASTER，对应的备份节点为BACKUP
   interface eth0 ## 绑定虚拟IP的网络接口，与本机IP地址所在的网络接口相同
   virtual_router_id 11 ## 虚拟路由的ID号，两个节点设置必须一样,建议用IP最后段
   mcast_src_ip 192.168.1.222 ## 本机 IP 地址
   priority 100 ## 节点优先级，值范围0-254，MASTER要比BACKUP高
   nopreempt ## 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
   advert_int 1 ## 组播信息发送间隔，两个节点设置必须一样，默认 1s
   ## 设置验证信息，两个节点必须一致
   authentication {
      auth_type PASS
      auth_pass 1111
   }
   ## 将 track_script 块加入 instance 配置块
      track_script {
      chk_nginx ## 执行 Nginx 监控的服务
   }
   ## 虚拟 IP 池, 两个节点设置必须一样
   virtual_ipaddress {
      192.168.1.220  ## 虚拟 ip，可以定义多个
   }
}


```

#### 备节点
```python
[root@nginx01 ~]# vim /etc/keepalived/keepalived.conf
global_defs {
  ##keepalived自带的邮件提醒需要开启sendmail服务。建议用独立的监控或第三方SMTP
  ##标识本节点的字条串，通常为 hostname
  router_id 192.168.1.221
}
##keepalived会定时执行脚本并对脚本执行的结果进行分析，动态调整vrrp_instance
   的优先级。如果脚本执行结果为0，并且weight配置的值大于0，则优先级相应的增加。
   如果脚本执行结果非0，并且weight配置的值小于 0，则优先级相应的减少。其他情况，
   维持原本配置的优先级，即配置文件中priority对应的值。
vrrp_script chk_nginx {
   script "/etc/keepalived/nginx_check.sh" ## 检测 nginx 状态的脚本路径
   interval 2 ## 检测时间间隔
   weight -20 ## 如果条件成立，权重-20
}
## 定义虚拟路由，VI_1为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
   state BAKUP ## 主节点为MASTER，对应的备份节点为BACKUP
   interface eth0 ## 绑定虚拟IP的网络接口，与本机IP地址所在的网络接口相同
   virtual_router_id 11 ## 虚拟路由的ID号，两个节点设置必须一样,建议用IP最后段
   mcast_src_ip 192.168.1.221 ## 本机 IP 地址
   priority 90 ## 节点优先级，值范围0-254，MASTER要比BACKUP高
   nopreempt ## 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
   advert_int 1 ## 组播信息发送间隔，两个节点设置必须一样，默认 1s
   ## 设置验证信息，两个节点必须一致
   authentication {
      auth_type PASS
      auth_pass 1111
   }
   ## 将 track_script 块加入 instance 配置块
      track_script {
      chk_nginx ## 执行 Nginx 监控的服务
   }
   ## 虚拟 IP 池, 两个节点设置必须一样
   virtual_ipaddress {
      192.168.1.220  ## 虚拟 ip，可以定义多个
   }
}
```





### 编写 Nginx 状态检测脚本 
```shell


[root@nginx02 ~]# vim /etc/keepalived/nginx_check.sh
[root@nginx01 ~]# vim /etc/keepalived/nginx_check.sh

!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi

```

---

### 高可用测试


**(1)关闭 192.168.1.222 中的 Nginx，Keepalived会将它重新启动**

```python
[root@nginx02 ~]# /usr/local/nginx/sbin/nginx -s stop
[root@nginx02 ~]# netstat -ntlp | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      73336/nginx
```

 
 
 
**(2)关闭 192.168.1.222 中的 Keepalived，VIP 会切换到 192.168.1.221 中**  

```python
[root@nginx02 ~]# service keepalived stop
Stopping keepalived:                                       [  OK  ]
[root@nginx02 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:de:c0:68 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.222/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::20c:29ff:fede:c068/64 scope link 
       valid_lft forever preferred_lft forever

 
 
 [root@nginx01 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:f5:7e:84 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.221/24 brd 192.168.1.255 scope global eth0
    inet 192.168.1.220/32 scope global eth0
    inet6 fe80::20c:29ff:fef5:7e84/64 scope link 
       valid_lft forever preferred_lft forever
```


**(3)重新启动 192.168.1.222 中的 Keepalived，VIP 又会切回到 192.168.1.222 中来**
 

```python
[root@nginx02 ~]# service keepalived start
Starting keepalived:                                       [  OK  ]
[root@nginx02 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:de:c0:68 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.222/24 brd 192.168.1.255 scope global eth0
    inet 192.168.1.220/32 scope global eth0
    inet6 fe80::20c:29ff:fede:c068/64 scope link 
       valid_lft forever preferred_lft forever
       
       
       
[root@nginx01 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:f5:7e:84 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.221/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::20c:29ff:fef5:7e84/64 scope link 
       valid_lft forever preferred_lft forever

```





