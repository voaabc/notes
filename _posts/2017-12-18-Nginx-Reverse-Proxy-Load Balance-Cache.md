---
layout: post
title: Nginx实现反向代理负载均衡与静态缓存
categories: Nginx
tags: 反向代理 负载均衡 静态缓存
---

* content
{:toc}

### 介绍
Nginx是一款轻量级的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。在连接高并发的情况下，Nginx是Apache服务器不错的替代品，能够支持高达50000个并发连接数的响应。

#### 反向代理
反向代理（Reverse Proxy）方式是指以**代理服务器**来接受internet上的连接请求，然后将**请求转发**给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为**一个服务器**。
##### 作用
- **保护网站安全**：任何来自Internet的请求都必须先经过代理服务器；
- **通过配置缓存功能加速Web请求**：可以缓存真实Web服务器上的某些静态资源，减轻真实Web服务器的负载压力；
- **实现负载均衡**：充当负载均衡服务器均衡地分发请求，平衡集群中各个服务器的负载压力；

#### 负载均衡
负载均衡，英文名称为Load Balance，其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。

#### Nginx缓存
nginx的http_proxy模块，可以实现类似于Squid的缓存功能。Nginx对客户已经访问过的内容在Nginx服务器本地建立副本，这样在一段时间内再次访问该数据，就不需要通过Ｎginx服务器再次向后端服务器发出请求，所以能够减少Ｎginx服务器与后端服务器之间的网络流量，减轻网络拥塞，同时还能减小数据传输延迟，提高用户访问速度。同时，当后端服务器宕机时，Nginx服务器上的副本资源还能够回应相关的用户请求，这样能够提高后端服务器的鲁棒性。



---
### 实验环境

Hostnane | IP  | 系统 | 规划
---|------|------|------
web01 | 192.168.1.223 | Centos 6.5 |  Web server
web02 | 192.168.1.224 | Centos 6.5 |  Web server
nginx02 | 192.168.1.221 | Centos 6.5 |  Nginx proxy

### 实验拓扑

```

A[Client]-->B{Nginx proxy}
B --> C[Web01]
B --> D[Web02]
```


---

### 实验步骤
#### 安装：（我们在这里使用编译安装）


```shell
[root@nginx02 ~]# yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
[root@nginx02 ~]# cd /setup/
[root@nginx02 setup]# tar -xzf nginx-1.8.0.tar.gz 
[root@nginx02 setup]# cd nginx-1.8.0
[root@nginx02 nginx-1.8.0]# ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  man  README  src
[root@nginx02 nginx-1.8.0]# ./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --user=nginx --group=nginx --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_flv_module --with-http_mp4_module --http-client-body-temp-path=/var/tmp/nginx/client --http-proxy-temp-path=/var/tmp/nginx/proxy --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi

---------------
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + md5: using OpenSSL library
  + sha1: using OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx configuration prefix: "/etc/nginx"
  nginx configuration file: "/etc/nginx/nginx.conf"
  nginx pid file: "/var/run/nginx/nginx.pid"
  nginx error log file: "/var/log/nginx/error.log"
  nginx http access log file: "/var/log/nginx/access.log"
  nginx http client request body temporary files: "/var/tmp/nginx/client"
  nginx http proxy temporary files: "/var/tmp/nginx/proxy"
  nginx http fastcgi temporary files: "/var/tmp/nginx/fastcgi"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
  
  ------------------

[root@nginx02 nginx-1.8.0]# make && make install
[root@nginx02 ~]# groupadd -r nginx
[root@nginx02 ~]# useradd -r -g nginx nginx
[root@nginx02 ~]# mkdir /var/tmp/nginx/{client,proxy,fastcgi} -p    <---创建编译安装时所需的目录
[root@nginx02 ~]# cd /etc/nginx/
[root@nginx02 nginx]# ls
fastcgi.conf          fastcgi_params.default  mime.types          nginx.conf.default   uwsgi_params
fastcgi.conf.default  koi-utf                 mime.types.default  scgi_params          uwsgi_params.default
fastcgi_params        koi-win                 nginx.conf          scgi_params.default  win-utf


```
#### Nginx的主配置文件介绍
Nginx的配置文件中参数较多，我主要说说重要的部分。


```c++
#user  nobody; #运行用户
worker_processes  1; #启动进程，通常设置成cpu数量减1
 
#全局错误日志及Pid文件
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#pid        logs/nginx.pid;
 
#工作模式及连接数上线
events {
    worker_connections  1024; #单个后台worker process进程的最大并发链接数
}
 
#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    include       mime.types; #设定mime类型，类型由mime.type文件定义
    default_type  application/octet-stream; #设定日志格式
 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
 
    #access_log  logs/access.log  main;
 
    #指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime
    sendfile        on; 
    #tcp_nopush     on;
 
    #keepalive_timeout  0; #连接超时时间
    keepalive_timeout  65;
 
    #gzip  on; #开启gzip压缩
 
    server {
        listen       80; #侦听80端口
        server_name  localhost; #定义使用localhost访问
 
        #charset koi8-r;
  
        #access_log  logs/host.access.log  main; #设定本虚拟机的访问日志
 
        #默认请求
        location / {
            root   html; #定义服务器的默认网站根目录位置
            index  index.html index.htm; #定义首页索引文件的名称
        }
 
        #error_page  404              /404.html; #定义错误页面
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html; #定义错误提示页面
        location = /50x.html {
            root   html;
        }
 
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
 
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #PHP脚本请求全部转发到fastcgi处理，使用fastcgi默认配置
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
 
 
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
    #基于https验证访问
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
 
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
 
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
 
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
```

#### 配置Nginx实现反向代理负载均衡

#定义一个upstream(负载均衡组)，组名为nginx02_proxy，在server组里直接调用组名

```java
http {

    upstream nginx02_proxy {
        server 192.168.1.223:80 weight=1 max_fails=2 fail_timeout=1;    <---两台real server为WEB的IP
        server 192.168.1.224:80 weight=1 max_fails=2 fail_timeout=1;
        #权重为1，1秒算超时连续2次超时说明检测失败
        }
 
    server {
        listen       80;
        server_name  nginx02;    <---修改为本机hostname
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            #重点关注
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #禁用缓存         
            proxy_buffering off;
            
            
            proxy_pass http://nginx02_proxy;    <---将对本服务器首页的请求代理至负载均衡组nginx02_proxy的两台real server
        }
    }
}
```


```shell
[root@nginx02 nginx]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@nginx02 nginx]# /usr/local/nginx/sbin/nginx
[root@nginx02 nginx]# netstat -ntlp | grep 80
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      3508/nginx
```




```java
#以下在web01上实现
[root@web01 ~]# yum install httpd -y
[root@web01 ~]# echo '<h1> real web server is web01</h1>' > /var/www/html/index.html

#以下在web02上实现
[root@web02 ~]# yum install httpd -y
[root@web02 ~]# echo '<h1> real web server is web01</h1>' > /var/www/html/index.html
[root@web02 ~]# yum -y install openssh-clients
[root@web02 ~]# service httpd start; ssh root@192.168.1.224 'service httpd start'   <---同时启动httpd
[root@web02 ~]# netstat -ntlp | grep 80; ssh root@192.168.1.223 netstat -ntlp | grep 80
tcp        0      0 :::80                       :::*                        LISTEN      1770/httpd          
The authenticity of host '192.168.1.223 (192.168.1.223)' can't be established.
RSA key fingerprint is 30:cd:ef:7c:15:10:2c:7d:c6:74:af:8c:c9:b3:57:27.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.223' (RSA) to the list of known hosts.
root@192.168.1.223's password: 
tcp        0      0 :::80                       :::*                        LISTEN      1710/httpd
```

#### 测试：对于两台Web Server负载均衡
http://192.168.1.222/

---

real web server is web01

---

real web server is web02


### 配置Nginx实现静态资源缓存


```shell
[root@nginx02 ~]# mkdir /cache/nginx -p 
[root@nginx02 ~]# chown nginx:nginx /cache/nginx/  <---将属主和属组都该为nginx
[root@nginx02 ~]# vim /etc/nginx/nginx.conf    <---添加以下参数

http {
    proxy_cache_path /cache/nginx/ levels=1:2 keys_zone=cache1:200m inactive=1d max_size=10g;
#设置Web缓存区名称为cache1，内存缓存空间大小为200MB，1天没有被访问的内容自动清除，硬盘缓存空间大小为30GB。levels=1:2 表示缓存目录的第一级目录是1个字符，第二级目录是2个字符，即/cache/nginx/a/1b这种形式

    server {
     location ~ \.(gif|jpg|jpeg|png|bmp|ico)$ {
            proxy_set_header Host  $host;
            proxy_set_header X-Forwarded-For  $remote_addr;
            proxy_pass http://192.168.1.224;
            proxy_cache cache1; #设置资源缓存的zone
            proxy_cache_key $host$uri$is_args$args; #设置缓存的key，以域名、URI、参数组合成Web缓存的Key值，Nginx根据Key值哈希，存储缓存内容到二级缓存目录内
            proxy_cache_valid 200 304 12h;  #对不同的HTTP状态码设置不同的缓存时间
            expires 7d; #缓存时间
        }
    
    }

        

[root@nginx02 ~]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@nginx02 ~]# /usr/local/nginx/sbin/nginx -s stop
[root@nginx02 ~]# /usr/local/nginx/sbin/nginx
```

**测试**
http://192.168.1.222/123.jpg

---

```python

[root@nginx02 ~]# cd /cache/nginx/
[root@nginx02 nginx]# ll
total 4
drwx------. 3 nginx nginx 4096 Dec 18 02:17 e
[root@nginx02 nginx]# cd e/62/
[root@nginx02 62]# ll
total 44
-rw-------. 1 nginx nginx 44137 Dec 18 02:17 4049f2f30e7026777a0ced24ca6db62e
```






