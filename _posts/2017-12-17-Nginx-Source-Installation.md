---
layout: post
title: Nginx源码安装
categories: Nginx
tags: Nginx



---

* content
{:toc}

### 一、下载Nginx源文件

进入nginx官网下载nginx的稳定版本，我下载的是1.10.0。
下载：wget http://nginx.org/download/nginx-1.10.0.tar.gz
解压：tar -zxvf nginx-1.10.0.tar.gz
 
### 二、检查安装依赖项

执行下面的命令安装nginx的依赖库：


```js
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```


 
### 三、配置Nginx安装选项

我这里只配置安装到/opt目录下，其它选项可执行./configure --help查看。
cd nginx安装目录，执行如下命令：


```js
./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx
```


 

官网参数配置说明：http://nginx.org/en/docs/configure.html

 
### 四、编译并安装


```java
[root@keepalived-nginx1 nginx-1.10.0]# make & make install
```


 

 
### 五、启动、停止、重启



```js
[root@keepalived-nginx1 ~]# nginx
[root@keepalived-nginx1 ~]# ps aux | grep nginx
root       3670  0.0  0.0  24432   828 ?        Ss   09:57   0:00 nginx: master process nginx
nobody     3671  0.0  0.1  24852  1408 ?        S    09:57   0:00 nginx: worker process
root       3673  0.0  0.0 103264   876 pts/0    S+   09:57   0:00 grep nginx
[root@keepalived-nginx1 ~]# nginx -s stop
[root@keepalived-nginx1 ~]# nginx
[root@keepalived-nginx1 ~]# nginx -s reload
```





nginx默认配置启动成功后，会有两个进程，一个主进程（守护进程），一个工作进程。主进程负责管理工作进程，工作进程负责处理用户的http请求。

 
### 六、配置nginx开机启动

将/usr/bin/nginx命令添加到/etc/rc.d/rc.local文件中，rc.local文件会在系统启动的时候执行。


```js
vim /etc/rc.d/rc.local
# 添加如下参数
/usr/bin/nginx

#CentOS7
chmod +x /etc/rc.d/rc.local
```


 





