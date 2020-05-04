---
layout: post
title: Docker容器常用命令
categories: Docker
tags: Docker  
---

* content
{:toc}

### 先更新软件包

   yum -y update

 
### 安装Docker虚拟机

yum install -y docker

 
### 运行、重启、关闭Docker虚拟机

service docker start

service docker start

service docker stop

 
### 搜索镜像

docker search 镜像名称

```shell
[root@docker01 ~]# docker search tomcat
INDEX       NAME                                    DESCRIPTION                                     STARS     OFFICIAL
docker.io   docker.io/tomcat                        Apache Tomcat is an open source implementa...   2407      [OK]

```
 
### 下载镜像

docker pull 镜像名称

```shell
[root@docker01 ~]# docker pull tomcat:8.5.32
Trying to pull repository docker.io/library/tomcat ... 
8.5.32: Pulling from docker.io/library/tomcat
55cbf04beb70: Pull complete 
1607093a898c: Pull complete 
9a8ea045c926: Pull complete 
1290813abd9d: Pull complete 
8a6b982ad6d7: Pull complete 
abb029e68402: Pull complete 
d068d0a738e5: Pull complete 
42ee47bb0c52: Pull complete 
ae9c861aed25: Pull complete 
60bba9d0dc8d: Pull complete 
15222e409530: Pull complete 
2dcc81b69024: Pull complete 
Digest: sha256:bbdb0de8298ab7281ff28331a9e4129562820ac54e243e44c3749f389876f562
Status: Downloaded newer image for docker.io/tomcat:8.5.32
```

### 查看镜像

docker images

```shell
[root@docker01 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/tomcat    8.5.32              5808f01b11bf        9 months ago        463 MB
```

### 删除镜像

docker rmi 镜像名称

 
### 运行容器

docker run 启动参数  镜像名称

* 使用命令：docker run --name container-name:tag -d image-name

1. --name：自定义容器名，不指定时，docker 会自动生成一个名称
2. -d：表示后台运行容器
3. image-name：指定运行的镜像名称以及 Tag 

* 如下所示启动 docker.io/tomcat 镜像成功，前缀 docker.io 可以不写，后面的 tag 版本号要指定。可以使用 docker ps 命令查看容器

```shell
[root@docker01 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/tomcat    8.5.32              5808f01b11bf        9 months ago        463 MB
[root@docker01 ~]# docker run --name myTomcat -d tomcat:8.5.32
687eddfaf1db351635e168d808a4497f39f94f8d17cbf349c2a4ff9587d6c41c
[root@docker01 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/tomcat    8.5.32              5808f01b11bf        9 months ago        463 MB
[root@docker01 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
687eddfaf1db        tomcat:8.5.32       "catalina.sh run"   26 seconds ago      Up 24 seconds       8080/tcp            myTomcat
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
687eddfaf1db        tomcat:8.5.32       "catalina.sh run"   29 seconds ago      Up 28 seconds       8080/tcp            myTomcat
```
 
### 查看容器列表

docker ps -a

```shell
[root@docker01 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
687eddfaf1db        tomcat:8.5.32       "catalina.sh run"   26 seconds ago      Up 24 seconds       8080/tcp            myTomcat
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
687eddfaf1db        tomcat:8.5.32       "catalina.sh run"   29 seconds ago      Up 28 seconds       8080/tcp            myTomcat
```


 


 
### 查看容器信息

docker inspect 容器ID


### 停止、挂起、恢复容器

> docker stop 容器ID
> docker pause 容器ID
> docker unpause 容器ID


 
### 删除容器

docker rm 容器ID

* 使用 docker rm container-id 命令 删除容器，删除容器前，必须先停止容器运行，根据 容器 id 进行删除
* rm 参数是删除容器，rmi 参数是删除镜像
* 镜像运行在容器中，docker 中可以运行多个互补干扰的容器，可以将同一个镜像在多个容器中进行运行

```shell
[root@docker01 ~]# docker stop myTomcat
myTomcat
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
687eddfaf1db        tomcat:8.5.32       "catalina.sh run"   44 minutes ago      Exited (143) 9 seconds ago                       myTomcat
[root@docker01 ~]# docker rm 687eddfaf1db
687eddfaf1db
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### 端口映射
* 使用命令：docker run --name container-name:tag -d -p 服务器端口:Docker 端口 image-name

1. --name：自定义容器名，不指定时，docker 会自动生成一个名称
2. -d：表示后台运行容器
3. image-name：指定运行的镜像名称以及 Tag 
4. -p 表示进行服务器与 Docker 容器的端口映射，默认情况下容器中镜像占用的端口是 Docker 容器中的端口与外界是隔绝的，必须进行端口映射才能访问

* 如下所示：服务器防火墙先开放了 8080、8090 端口，否则防火墙不开放端口的话，从其它电脑也是无法访问服务器的
* 然后 运行了 两个容器，容器名称分别指定为 "myTomcat1"、"myTomcat2"、两个容器中都是同一个 docker.io/tomcat:8.5.32 镜像
* 两个容器都指定了端口映射，分别是8080、8090 ，都会转发到 Docker 容器内部
*启动成功之后，ip addr show 查一下服务器 ip 地址，然后就能从物理机上访问了

```shell
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker01 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/tomcat    8.5.32              5808f01b11bf        9 months ago        463 MB
[root@docker01 ~]# docker run --name myTtomcat1 -d -p 8080:8080 tomcat:8.5.32
fdf6c3af75bf121035b1aa67926e2d6dd287c6f755583128db8460d2d953d908
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
fdf6c3af75bf        tomcat:8.5.32       "catalina.sh run"   14 seconds ago      Up 13 seconds       0.0.0.0:8080->8080/tcp   myTtomcat1
[root@docker01 ~]# docker run --name myTtomcat2 -d -p 8090:8080 tomcat:8.5.32
6149e23a6822a6215c81e217d77446112ef19d78c367ec0844b39e89f523274c
[root@docker01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
6149e23a6822        tomcat:8.5.32       "catalina.sh run"   5 seconds ago       Up 5 seconds        0.0.0.0:8090->8080/tcp   myTtomcat2
fdf6c3af75bf        tomcat:8.5.32       "catalina.sh run"   30 seconds ago      Up 29 seconds       0.0.0.0:8080->8080/tcp   myTtomcat1
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
    inet6 fe80::c71:19f9:8df7:f491/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:08:94:f1:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:8ff:fe94:f103/64 scope link 
       valid_lft forever preferred_lft forever
9: veth8b3ccda@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether f6:03:66:25:66:bf brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::f403:66ff:fe25:66bf/64 scope link 
       valid_lft forever preferred_lft forever
11: vethcebb16c@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether e6:2c:c9:82:16:dd brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::e42c:c9ff:fe82:16dd/64 scope link 
       valid_lft forever preferred_lft forever

```

### 容器日志

* 使用 docker logs container-name/container-id 命令 可以查看容器日志信息，指定容器名或者 容器 id 即可

 
### 数据卷管理

docker volume create 数据卷名称  #创建数据卷
docker volume rm 数据卷名称  #删除数据卷
docker volume inspect 数据卷名称  #查看数据卷

 
### 网络管理

docker network ls 查看网络信息
docker network create --subnet=网段 网络名称
docker network rm 网络名称

 
### 避免VM虚拟机挂起恢复之后，Docker虚拟机断网

    vim /etc/sysctl.conf


文件中添加net.ipv4.ip_forward=1这个配置
重启网络服务
   systemctl  restart network



