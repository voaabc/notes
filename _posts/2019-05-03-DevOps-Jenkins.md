---
layout: post
title: DevOps之Jenkins部署
categories: DevOps Jenkins
tags: DevOps Jenkins
---

* content
{:toc}


### 准备linux初始环境

```shell
[root@jenkins ~]# systemctl stop firewalld
[root@jenkins ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@jenkins ~]# sed -i "s#SELINUX=enforcing#SELINUX=disabled#g" /etc/sysconfig/selinux
[root@jenkins ~]# setenforce 0
[root@jenkins ~]# getenforce
Permissive

[root@jenkins ~]# yum install vim wget -y
[root@jenkins ~]# yum install -y java
[root@jenkins ~]# java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
[root@jenkins ~]# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
[root@jenkins ~]# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
[root@jenkins ~]# yum list | grep 'jenkins'
jenkins.noarch                              2.176.2-1.1                jenkins 

```

### 安装Jenkins

```shell
[root@jenkins ~]# yum install -y jenkins
# 创建Jenkins系统服务用户
[root@jenkins ~]# useradd deploy
[root@jenkins ~]# cp /etc/sysconfig/jenkins{,.bak}
[root@jenkins ~]# vim /etc/sysconfig/jenkins
# 大约在29行，改为deploy用户
29 JENKINS_USER="deploy"
# 确定Jenkins端口号8080
56 JENKINS_PORT="8080"
更改目录权限
[root@jenkins ~]# chown -R deploy:deploy /var/lib/jenkins
[root@jenkins ~]# chown -R deploy:deploy /var/log/jenkins/
启动Jenkins
[root@jenkins ~]# systemctl start jenkins
[root@jenkins ~]# lsof -i:8080
# 这里发现端口没起来，查看日志发现
[root@jenkins ~]# cat /var/log/jenkins/jenkins.log
java.io.FileNotFoundException: /var/cache/jenkins/war/META-INF/MANIFEST.MF (Permission denied)
# 然后赋予deploy目录权限
[root@jenkins ~]# chown -R deploy:deploy /var/cache/jenkins/
[root@jenkins ~]# systemctl restart jenkins
[root@jenkins ~]# lsof -i:8080
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    27455 deploy  164u  IPv6  76989      0t0  TCP *:webcache (LISTEN)

```




