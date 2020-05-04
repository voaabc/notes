---
layout: post
title: 使用Docker部署Zabbix
categories: Docker Zabbix
tags: Docker Zabbix
---

* content
{:toc}


#### 安装docker-compose

```shell
# curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
# docker-compose --version
docker-compose version 1.23.1, build b02f1306

```



#### 编写zabbix.yml
```shell
# mkdir /zabbix
# cd /zabbix/
/zabbix]# vim zabbix.yml
version: '3'
services: 
  mysql: 
    image: swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-mysql
    environment: 
      MYSQL_USER: zabbix
      MYSQL_DATABASE: zabbix
      MYSQL_PASSWORD: zabbix
      MYSQL_ROOT_PASSWORD: Sowhat?
    volumes: 
      - /data/mysql/zabbix:/var/lib/mysql
    ports:
      - 3306:3306
    restart: always
    networks: 
      - zabbix
  
  zabbix-java-gateway:
    image: swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-java-gateway
    ports: 
      - 10052:10052
    restart: always
    networks: 
      - zabbix

  zabbix-server: 
    image: swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-server
    environment:
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix
      MYSQL_ROOT_PASSWORD: Sowhat?
    links: 
      - mysql
    ports: 
      - 10051:10051
    depends_on: 
      - mysql
    restart: always
    networks: 
      - zabbix

  zabbix-web: 
    image: swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-web
    environment:
      PHP_TZ: Asia/Shanghai
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix
      MYSQL_ROOT_PASSWORD: Sowhat?
    links:
      - mysql   
    ports: 
      - 80:80
    depends_on: 
      - zabbix-server
      - mysql
    restart: always
    networks: 
      - zabbix

networks: 
  zabbix:
    driver: bridge



```




#### 启动容器组
```shell
[root@docker02 zabbix]# docker-compose -f zabbix.yml up -d --build
Creating network "zabbix_zabbix" with driver "bridge"
Pulling zabbix-java-gateway (swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-java-gateway:)...
Trying to pull repository swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-java-gateway ... 
latest: Pulling from swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-java-gateway
c67f3896b22c: Pull complete
28bd08666da4: Pull complete
21b8ee686afa: Pull complete
824932b5f2bc: Pull complete
8684d0f43c47: Pull complete
e29aa4d7f4bf: Pull complete
b908f38442f8: Pull complete
2384821aec7a: Pull complete
Digest: sha256:fc95fd77ab38768b0d35101b19622c73d1aa7f0fdda0e4c82699f15c4a1dfa8c
Status: Downloaded newer image for swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-java-gateway:latest
Pulling mysql (swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-mysql:)...
Trying to pull repository swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-mysql ... 
latest: Pulling from swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-mysql
a5a6f2f73cd8: Pull complete
936836019e67: Pull complete
283fa4c95fb4: Pull complete
1f212fb371f9: Pull complete
e2ae0d063e89: Pull complete
5ed0ae805b65: Pull complete
0283dc49ef4e: Pull complete
a7905d9fbbea: Pull complete
cd2a65837235: Pull complete
5f906b8da5fe: Pull complete
e81e51815567: Pull complete
423a0fdeb46c: Pull complete
Digest: sha256:74a62f652e6cc5ba4ac8ec28e55227c781b5b1bf5d8533597e4200353f2c00ed
Status: Downloaded newer image for swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-mysql:latest
Pulling zabbix-server (swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-server:)...
Trying to pull repository swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-server ... 
latest: Pulling from swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-server
d6a5679aa3cf: Pull complete
7a046b910a3c: Pull complete
497545b1664f: Pull complete
58ac09628c71: Pull complete
9e045e3ec3a3: Pull complete
432e63b42dc5: Pull complete
Digest: sha256:69d854834e344bb512d8f70cbeaf2226db5db5b19c926545fc598e04e0d1156c
Status: Downloaded newer image for swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-server:latest
Pulling zabbix-web (swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-web:)...
Trying to pull repository swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-web ... 
latest: Pulling from swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-web
c67f3896b22c: Already exists
90884e24b31c: Pull complete
c9ddded2780e: Pull complete
1fb53af6cb01: Pull complete
fbefe3595fbc: Pull complete
6b4acce82f87: Pull complete
a4bf8619ae81: Pull complete
604f32616982: Pull complete
1bf0ca328b88: Pull complete
913cf2a0cff6: Pull complete
571ab96e2d5b: Pull complete
e6e318768910: Pull complete
ad8b7746cfb4: Pull complete
76a428e15158: Pull complete
Digest: sha256:6435ee08428960c7f8385e783b776c15eb2f5bcc4fc4671fb40eabd95da204d8
Status: Downloaded newer image for swr.cn-north-1.myhuaweicloud.com/rj-bai/zabbix-web:latest
Creating zabbix_zabbix-java-gateway_1 ... done
Creating zabbix_mysql_1               ... done
Creating zabbix_zabbix-server_1       ... done
Creating zabbix_zabbix-web_1          ... done



[root@docker02 zabbix]# docker-compose -f zabbix.yml ps
            Name                         Command               State                 Ports           
-----------------------------------------------------------------------------------------------------
zabbix_mysql_1                 docker-entrypoint.sh mysqld   Restarting                              
zabbix_zabbix-java-gateway_1   docker-entrypoint.sh          Up           0.0.0.0:10052->10052/tcp   
zabbix_zabbix-server_1         docker-entrypoint.sh          Up           0.0.0.0:10051->10051/tcp   
zabbix_zabbix-web_1            docker-entrypoint.sh          Up           443/tcp, 0.0.0.0:80->80/tcp


[root@docker02 lib]# docker logs -f zabbix_mysql_1
chown: cannot read directory '/var/lib/mysql/': Permission denied
chown: cannot read directory '/var/lib/mysql/': Permission denied
chown: cannot read directory '/var/lib/mysql/': Permission denied
chown: cannot read directory '/var/lib/mysql/': Permission denied
chown: cannot read directory '/var/lib/mysql/': Permission denied
chown: cannot read directory '/var/lib/mysql/': Permission denied


[root@docker02 zabbix]# sed -i s#SELINUX=enforcing#SELINUX=disabled# /etc/selinux/config
[root@docker02 zabbix]# more /etc/selinux/config 

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


[root@docker02 zabbix]# setenforce 0

[root@docker02 zabbix]# docker-compose -f zabbix.yml ps
            Name                         Command             State                 Ports              
------------------------------------------------------------------------------------------------------
zabbix_mysql_1                 docker-entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp, 33060/tcp
zabbix_zabbix-java-gateway_1   docker-entrypoint.sh          Up      0.0.0.0:10052->10052/tcp         
zabbix_zabbix-server_1         docker-entrypoint.sh          Up      0.0.0.0:10051->10051/tcp         
zabbix_zabbix-web_1            docker-entrypoint.sh          Up      443/tcp, 0.0.0.0:80->80/tcp

```



#### 安装zabbix-agent

```shell

#rpm -ivh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
#yum -y install zabbix-agent-4.0.1
[root@docker02 zabbix]# docker exec -it zabbix_zabbix-server_1  ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
61: eth0@if62: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:14:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.20.0.4/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe14:4/64 scope link 
       valid_lft forever preferred_lft forever


[root@docker02 zabbix]# vim /etc/zabbix/zabbix_agentd.conf
Server=172.18.0.4
UnsafeUserParameters=1
Include=/etc/zabbix/zabbix_agentd.d/*.conf


[root@docker02 zabbix]# systemctl start zabbix-agent.service

```




