---
layout: post
title: DevOps之GitLab部署
categories: DevOps GitLab
tags: DevOps GitLab
---

* content
{:toc}


#### 准备linux初始环境


```shell
[root@gitlab ~]# systemctl stop firewalld
[root@gitlab ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@gitlab ~]# sed -i "s#SELINUX=enforcing#SELINUX=disabled#g" /etc/sysconfig/selinux
[root@gitlab ~]# more /etc/sysconfig/selinux 

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


[root@gitlab ~]# setenforce 0
[root@gitlab ~]# getenforce
Permissive

#安装gitlab依赖软件
[root@gitlab ~]# yum install curl policycoreutils openssh-server openssh-clients postfix

#下载gitlab yum仓库源
[root@gitlab ~]# curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

#启动postfix邮件服务
[root@gitlab ~]# systemctl start postfix
[root@gitlab ~]# systemctl enable  postfix

```
#### 安装gitlab

```shell
[root@gitlab ~]# yum -y install gitlab-ce

```

#### 手动配置ssl证书


```shell
[root@gitlab ~]# mkdir -p /etc/gitlab/ssl
[root@gitlab ~]# openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key" 2048

[root@gitlab ~]# openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:bj
Locality Name (eg, city) [Default City]:bj
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:gitlab.example.com
Email Address []:voaabc@163.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123456
An optional company name []:
```

```shell
[root@gitlab ~]# cd /etc/gitlab/ssl/
#ssl密钥和证书
[root@gitlab ssl]# ll
total 8
-rw-r--r--. 1 root root 1070 Jul 20 11:44 gitlab.example.com.csr
-rw-r--r--. 1 root root 1679 Jul 20 03:15 gitlab.example.com.key

#利用ssl密钥和证书创建签署证书
[root@gitlab ssl]# openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"
Signature ok
subject=/C=cn/ST=bj/L=bj/O=Default Company Ltd/CN=gitlab.example.com/emailAddress=voaabc@163.com
Getting Private key
[root@gitlab ssl]# ll
total 12
-rw-r--r--. 1 root root 1273 Jul 20 11:47 gitlab.example.com.crt
-rw-r--r--. 1 root root 1070 Jul 20 11:44 gitlab.example.com.csr
-rw-r--r--. 1 root root 1679 Jul 20 03:15 gitlab.example.com.key

#利用openssl签署pem 证书
[root@gitlab ssl]# openssl dhparam -out /etc/gitlab/ssl/dhparams.pem  2048
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
...........................................+...+..
[root@gitlab ssl]# ll
total 16
-rw-r--r--. 1 root root  424 Jul 20 11:47 dhparams.pem
-rw-r--r--. 1 root root 1273 Jul 20 11:47 gitlab.example.com.crt
-rw-r--r--. 1 root root 1070 Jul 20 11:44 gitlab.example.com.csr
-rw-r--r--. 1 root root 1679 Jul 20 03:15 gitlab.example.com.key

#更改ssl下的所有证书权限
[root@gitlab ssl]# chmod 600 *

#配置证书到gitlab配置文件中
[root@gitlab ssl]# vim /etc/gitlab/gitlab.rb
external_url 'https://gitlab.example.com'
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"

#初始化gitlab相关服务配置
[root@gitlab ssl]# gitlab-ctl reconfigure

#找到gitlab下的ningx反向代理工具
[root@gitlab ssl]# vim /var/opt/gitlab/nginx/conf/gitlab-http.conf
server {
  listen *:80;

  server_name gitlab.example.com;
  rewrite ^(.*)$ https://$host$1 permanent;

#重启gitlab使服务生效
[root@gitlab ssl]# gitlab-ctl restart
ok: run: alertmanager: (pid 29566) 0s
ok: run: gitaly: (pid 29576) 0s
ok: run: gitlab-monitor: (pid 29594) 0s
ok: run: gitlab-workhorse: (pid 29596) 1s
ok: run: grafana: (pid 29612) 0s
ok: run: logrotate: (pid 29623) 1s
ok: run: nginx: (pid 29630) 0s
ok: run: node-exporter: (pid 29636) 1s
ok: run: postgres-exporter: (pid 29641) 0s
ok: run: postgresql: (pid 29729) 0s
ok: run: prometheus: (pid 29740) 0s
ok: run: redis: (pid 29751) 0s
ok: run: redis-exporter: (pid 29783) 0s
ok: run: sidekiq: (pid 29792) 0s
ok: run: unicorn: (pid 29804) 1s

```






#### 验证
```shell
[root@gitlab ~]# git config --global user.name 'root'
[root@gitlab ~]# git config --global  --list
user.name=root

[root@gitlab ~]# mkdir test
[root@gitlab ~]# cd test/
[root@gitlab test]# vim /etc/hosts
[root@gitlab test]# git -c http.sslVerify=false clone https://gitlab.example.com/root/test1.git
Cloning into 'test1'...
Username for 'https://gitlab.example.com': root
Password for 'https://root@gitlab.example.com': 
warning: You appear to have cloned an empty repository.

[root@gitlab test]# cd test1/
[root@gitlab test1]# vim test1.py
[root@gitlab test1]# ls
test1.py
[root@gitlab test1]# pwd
/root/test/test1
[root@gitlab test1]# git add .
[root@gitlab test1]# git commit -m"First commit"

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account s default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'root@gitlab.(none)')

[root@gitlab test1]# git config --global user.email "admin@example.com"
[root@gitlab test1]# git config --global user.name "admin"
[root@gitlab test1]# git commit -m"First commit"
[master (root-commit) 907bb9c] First commit
 1 file changed, 1 insertion(+)
 create mode 100644 test1.py

[root@gitlab test1]# git -c http.sslVerify=false push origin master
Username for 'https://gitlab.example.com': root
Password for 'https://root@gitlab.example.com': 
Counting objects: 3, done.
Writing objects: 100% (3/3), 211 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://gitlab.example.com/root/test1.git
 * [new branch]      master -> master
```




```shell




```


