---
layout: post
title: Django商城项目开发(一) 
categories: Python Django
tags: Python Django
---

* content
{:toc}


## 项目架构的程序设计

### 项目使用技术

    基于Python语言，版本：>=3.5及以上。
    使用Django框架，版本：1.11.11的LTS版本。
    MySQL数据库，版本：
    连接数据库：pymysql=0.8.0
    图像处理： Pillow=5.0.0
    Web前端技术：HTML、CSS、JavaScript和Jquery等


### Python&Django虚拟环境

```sh
[root@centostest ~]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
[root@centostest ~]# yum install ntpdate wget vim tree -y
[root@centostest ~]# ntpdate -u  ntp.aliyun.com
[root@centostest ~]# wget  https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz
[root@centostest ~]# tar -xzvf Python-3.6.2.tgz -C /opt/
[root@centostest ~]# cd /opt/Python-3.6.2/
[root@centostest Python-3.6.2]# yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
[root@centostest Python-3.6.2]# ./configure --prefix=/opt/python36
[root@centostest Python-3.6.2]# make && make install
[root@centostest Python-3.6.2]# vim /etc/profile
export PATH=$PATH:/opt/python362/bin

[root@centostest Python-3.6.2]# python3
-bash: python3: command not found
[root@centostest Python-3.6.2]# source /etc/profile
[root@centostest Python-3.6.2]# python3
Python 3.6.2 (default, Jan 10 2020, 02:08:50) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
[root@centostest Python-3.6.2]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple virtualenv
Collecting virtualenv
  Downloading https://pypi.tuna.tsinghua.edu.cn/packages/05/f1/2e07e8ca50e047b9cc9ad56cf4291f4e041fa73207d000a095fe478abf84/virtualenv-16.7.9-py2.py3-none-any.whl (3.4MB)
    100% |████████████████████████████████| 3.4MB 401kB/s 
Installing collected packages: virtualenv
Successfully installed virtualenv-16.7.9
You are using pip version 9.0.1, however version 19.3.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
[root@centostest Python-3.6.2]# cd  /opt
[root@centostest opt]# virtualenv --no-site-packages --python=python3 env_1

--no-site-packages：表示使用一个只有Python3的环境，而不导入原来Python3中安装模块。
--python=python3：指定要被虚拟的解释器环境。
env_1：表示虚拟的Python环境目录。

Running virtualenv with interpreter /opt/python362/bin/python3
Already using interpreter /opt/python362/bin/python3
Using base prefix '/opt/python362'
New python executable in /opt/env_1/bin/python3
Also creating executable in /opt/env_1/bin/python
Installing setuptools, pip, wheel...
done.
[root@centostest opt]# source env_1/bin/activate
创建好虚拟环境后，需要激活虚拟目录

(env_1) [root@centostest opt]# tree -L 1 env_1/
env_1/
├── bin
├── include
└── lib

3 directories, 0 files
(env_1) [root@centostest opt]# tree env_1/bin/
env_1/bin/
├── activate
├── activate.csh
├── activate.fish
├── activate.ps1
├── activate_this.py
├── activate.xsh
├── easy_install
├── easy_install-3.6
├── pip
├── pip3
├── pip3.6
├── python -> python3
├── python3
├── python3.6 -> python3
├── python-config
└── wheel

0 directories, 16 files

(env_1) [root@centostest opt]# pip3 list
Package    Version
---------- -------
pip        19.3.1 
setuptools 44.0.0 
wheel      0.33.6

(env_1) [root@centostest opt]# python -V
Python 3.6.2

(env_1) [root@centostest opt]# deactivate
[root@centostest ~]# source /opt/env_1/bin/activate
(env_1) [root@centostest ~]# deactivate


[root@centostest ~]# source /opt/env_1/bin/activate
(env_1) [root@centostest ~]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django==1.11.11
(env_1) [root@centostest ~]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pymysql==0.8.0
(env_1) [root@centostest ~]# pip3 list
Package    Version
---------- -------
Django     1.11.11
Pillow     5.0.0  
pip        19.3.1 
PyMySQL    0.8.0  
pytz       2019.3 
setuptools 44.0.0 
wheel      0.33.6

```

### Docker&MySQL部署

```sh
yum remove docker docker-client docker-client-latest docker-common   docker-latest docker-latest-logrotate docker-logrotate docker-selinux  docker-engine-selinux docker-engine

yum install -y yum-utils   device-mapper-persistent-data   lvm2

#yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum makecache fast

yum list docker-ce --showduplicates | sort -r

#yum install docker-ce-18.03.1.ce

yum install docker-ce

service docker start
systemctl enable docker 
systemctl stop docker
systemctl start docker
docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40


systemctl status docker


(env_1) [root@centostest ~]# mkdir -p /opt/docker-mysql/conf.d
(env_1) [root@centostest ~]# vim /opt/docker-mysql/conf.d/config-file.cnf
[mysqld]
# 表名不区分大小写
lower_case_table_names=1 
#server-id=1
datadir=/var/lib/mysql
#socket=/var/lib/mysql/mysqlx.sock
#symbolic-links=0
# sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid


(env_1) [root@centostest ~]# mkdir -p /opt/docker-mysql/var/lib/mysql

(env_1) [root@centostest ~]# docker run --name mysql \
>     --restart=always \
>     -p 3306:3306 \
>     -v /opt/docker-mysql/conf.d:/etc/mysql/conf.d \
>     -v /opt/docker-mysql/var/lib/mysql:/var/lib/mysql \
>     -e MYSQL_ROOT_PASSWORD=123456-abc \
>     -d mysql:8.0.16
6e3920ff73522bec641c016d5f359d4dce45fd5533b8685e33a061e7daab271f

(env_1) [root@centostest ~]# yum install netstat -y
(env_1) [root@centostest ~]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      9308/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      10719/master        
tcp6       0      0 :::3306                 :::*                    LISTEN      63886/docker-proxy  
tcp6       0      0 :::22                   :::*                    LISTEN      9308/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      10719/master

参考：https://www.jianshu.com/p/d6febf6f95e0


systemctl stop firewalld
systemctl disable firewalld
```





## 基于Django框架的项目搭建

### 创建数据库 shopdb

```sql
(env_1) [root@centostest ~]# more /tmp/shopdb.sql 

-- 设置UTF8
set character set utf8;

-- 会员信息表（后台管理员信息也在此标准，通过状态区分）
CREATE TABLE `users`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(32) NOT NULL,
  `name` varchar(16) DEFAULT NULL,
  `password` char(32) NOT NULL,
  `sex` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `address` varchar(255) DEFAULT NULL,
  `code` char(6) DEFAULT NULL,
  `phone` varchar(16) DEFAULT NULL,
  `email` varchar(50) DEFAULT NULL,
  `state` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `addtime` datetime DEFAULT NULL, 
  PRIMARY KEY (`id`),      
  UNIQUE KEY `username` (`username`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 商品类别表
CREATE TABLE `type`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `pid` int(11) unsigned DEFAULT '0',
  `path` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 商品信息表
CREATE TABLE `goods`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `typeid` int(11) unsigned NOT NULL,
  `goods` varchar(32) NOT NULL,
  `company` varchar(50) DEFAULT NULL,
  `content` text,
  `price` double(6,2) unsigned NOT NULL,
  `picname` varchar(255) DEFAULT NULL,
  `store` int(11) unsigned NOT NULL DEFAULT '0', 
  `num` int(11) unsigned NOT NULL DEFAULT '0',
  `clicknum` int(11) unsigned NOT NULL DEFAULT '0',
  `state` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `addtime` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `typeid` (`typeid`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 订单信息表
CREATE TABLE `orders`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) unsigned DEFAULT NULL,
  `linkman` varchar(32) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  `code` char(6) DEFAULT NULL,
  `phone` varchar(16) DEFAULT NULL,
  `addtime` datetime DEFAULT NULL,
  `total` double(8,2) unsigned DEFAULT NULL,
  `state` tinyint(1) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 订单信息详情表
CREATE TABLE `detail`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `orderid` int(11) unsigned DEFAULT NULL,
  `goodsid` int(11) unsigned DEFAULT NULL,
  `name` varchar(32) DEFAULT NULL,
  `price` double(6,2) DEFAULT NULL,
  `num` int(11) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


-- 在user上表中添加一条后台管理员账户数据
insert into users values(null,'admin','管理员',md5('admin'),1,'北京市中南海','1000','1888888888','10000@qq.com',0,now())
```


```sh
(env_1) [root@centostest ~]# docker ps -a
\CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
6e3920ff7352        mysql:8.0.16        "docker-entrypoint.s…"   27 minutes ago      Up 27 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
(env_1) [root@centostest ~]# docker cp /tmp/shopdb.sql mysql:/tmp/

(env_1) [root@centostest ~]# docker exec -it mysql bash
root@6e3920ff7352:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.16 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE shopdb;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shopdb             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> USE shopdb;
Database changed
mysql> source /tmp/shopdb.sql
Query OK, 0 rows affected, 1 warning (0.01 sec)

Query OK, 0 rows affected, 1 warning (0.01 sec)

Query OK, 0 rows affected, 1 warning (0.00 sec)

Query OK, 0 rows affected, 1 warning (0.00 sec)

Query OK, 0 rows affected, 1 warning (0.01 sec)

Query OK, 1 row affected (0.01 sec)



mysql> use mysql;
Database changed
mysql> select user, host, plugin from user where user='root';
+------+-----------+-----------------------+
| user | host      | plugin                |
+------+-----------+-----------------------+
| root | %         | caching_sha2_password |
| root | localhost | caching_sha2_password |
+------+-----------+-----------------------+
2 rows in set (0.00 sec)

mysql> CREATE USER 'shop'@'%' IDENTIFIED WITH mysql_native_password BY '123456a!';
Query OK, 0 rows affected (0.01 sec)

mysql> select user, host, plugin from user where user='shop';
+------+------+-----------------------+
| user | host | plugin                |
+------+------+-----------------------+
| shop | %    | mysql_native_password |
+------+------+-----------------------+
1 row in set (0.00 sec)


mysql> grant all privileges on shopdb.* to 'shop'@'%' with grant option;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges; 
Query OK, 0 rows affected (0.01 sec)

mysql> show grants for shop;
+--------------------------------------------------------------------+
| Grants for shop@%                                                  |
+--------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `shop`@`%`                                   |
| GRANT ALL PRIVILEGES ON `shopdb`.* TO `shop`@`%` WITH GRANT OPTION |
+--------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> show variables like 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8                           |
| character_set_connection | utf8mb4                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8                           |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8                           |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.01 sec)

```


### 创建项目 myobject 框架和应用 myamdin、web和common


```sh
# 创建项目框架 `myobject`
(env_1) [root@centostest opt]# django-admin startproject myobject
(env_1) [root@centostest opt]# ll
total 4
drwx--x--x.  4 root root   28 Jan 14 04:07 containerd
drwxr-xr-x.  4 root root   31 Jan 14 04:18 docker-mysql
drwxr-xr-x.  5 root root   43 Jan 10 02:23 env_1
drwxr-xr-x.  3 root root   39 Jan 14 05:04 myobject
drwxr-xr-x.  6 root root   56 Jan 10 02:10 python362
drwxr-xr-x. 17  501  501 4096 Jan 10 02:09 Python-3.6.2
(env_1) [root@centostest opt]# cd myobject
# 在项目中创建一个myadmin应用(项目的后台管理)
(env_1) [root@centostest myobject]# python manage.py startapp myadmin
# 在项目中再创建一个web应用(项目前台)
(env_1) [root@centostest myobject]# python manage.py startapp web
# 在项目中再创建一个common应用(项目的前台和后台的公告应用)
(env_1) [root@centostest myobject]# python manage.py startapp common
# 创建模板目录
(env_1) [root@centostest myobject]# mkdir templates
(env_1) [root@centostest myobject]# mkdir templates/myadmin
(env_1) [root@centostest myobject]# mkdir templates/web
# 创建静态资源目录
(env_1) [root@centostest myobject]# mkdir static
(env_1) [root@centostest myobject]# mkdir static/myadmin
(env_1) [root@centostest myobject]# mkdir static/web
# 创建前后台应用模板目录,并在里面各创建一个`__init__.py`和`index.py`的空文件
(env_1) [root@centostest myobject]# mkdir myadmin/views
(env_1) [root@centostest myobject]# touch myadmin/views/__init__.py
(env_1) [root@centostest myobject]# touch myadmin/views/index.py
(env_1) [root@centostest myobject]# mkdir web/views
(env_1) [root@centostest myobject]# touch web/views/__init__.py
(env_1) [root@centostest myobject]# touch web/views/index.py
# 删除前后台应用的默认模板文件
(env_1) [root@centostest myobject]# rm -rf myadmin/views.py
(env_1) [root@centostest myobject]# rm -rf web/views.py
# 拷贝路由文件到应用目录中
(env_1) [root@centostest myobject]# cp myobject/urls.py   myadmin/urls.py
(env_1) [root@centostest myobject]# cp myobject/urls.py   web/urls.py
(env_1) [root@centostest myobject]# cd ..
(env_1) [root@centostest opt]# tree myobject
myobject
├── common
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── myadmin
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views
│       ├── index.py
│       └── __init__.py
├── myobject
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   └── settings.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── static
│   ├── myadmin
│   └── web
├── templates
│   ├── myadmin
│   └── web
└── web
    ├── admin.py
    ├── apps.py
    ├── __init__.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    ├── urls.py
    └── views
        ├── index.py
        └── __init__.py

16 directories, 32 files

```

### 项目框架配置

```python
#编辑myobject/myobject/__init__.py文件，添加Pymysql的数据库操作支持

import pymysql
pymysql.install_as_MySQLdb()



#编辑myobject/myobject/settings.py文件
# 1. 配置允许访问的主机名信息
ALLOWED_HOSTS = ['*']
或
ALLOWED_HOSTS = ['localhost','127.0.0.1','192.168.2.240']

...

# 2. 将myadmin和web的应用添加到项目框架结构中
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myadmin',
    'web',
    'common',
]

...

# 3. 配置模板目录 os.path.join(BASE_DIR,'templates')
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

...

# 4. 配置项目的数据库连接信息：
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'shopdb',
        'USER': 'shop',
        'PASSWORD': '123456a!',
        'HOST': '192.168.220.131',
        'PORT': '3306',
    }
}

...

# 5. 设置时区和语言 
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

...

# 6. 配置网站的静态资源目录
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]



```

### 项目urls路由信息配置

```python


# 打开根路由文件：myobject/myobject/urls.py路由文件，编写路由配置信息

# myobject/myobject/urls.py

from django.conf.urls import url,include
#from django.contrib import admin

urlpatterns = [
#   url(r'^admin/', admin.site.urls),
    url(r'^myadmin/', include('myadmin.urls')), #网站后台路由
    url(r'^', include('web.urls')),           #网站前台路由
]

# 打开项目后台管理路由文件：myobject/myadmin/urls.py路由文件，编写路由配置信息

# myobject/myadmin/urls.py

from django.conf.urls import url

from myadmin.views import index

urlpatterns = [
    # 后台首页
    url(r'^$', index.index, name="myadmin_index"),

]

# 打开项目前台路由文件：myobject/web/urls.py路由文件，编写路由配置信息

# myobject/web/urls.py

from django.conf.urls import url

from web.views import index

urlpatterns = [
    url(r'^$', index.index, name="index"),
]





```





### 编写后台视图测试

```python

#编辑后台视图文件

# myobject/myadmin/views/index.py
from django.shortcuts import render
from django.http import HttpResponse

#后台首页
def index(request):
    return HttpResponse('欢迎进入商城网站后台！')

# myobject/web/views/index.py
from django.shortcuts import render
from django.http import HttpResponse

#前台首页
def index(request):
    return HttpResponse('欢迎进入商城网站前台首页！')

#运行测试
#在项目根目录下启动服务，并使用浏览器访问测试：http://192.168.220.131:8000/myadmin/

(env_1) [root@centostest ~]# cd /opt/myobject/
(env_1) [root@centostest myobject]# ll
total 4
drwxr-xr-x. 4 root root 142 Jan 14 22:24 common
-rwxr-xr-x. 1 root root 806 Jan 14 05:04 manage.py
drwxr-xr-x. 5 root root 154 Jan 14 22:24 myadmin
drwxr-xr-x. 3 root root  93 Jan 14 05:04 myobject
drwxr-xr-x. 4 root root  32 Jan 14 05:05 static
drwxr-xr-x. 4 root root  32 Jan 14 05:05 templates
drwxr-xr-x. 5 root root 154 Jan 14 22:24 web
(env_1) [root@centostest myobject]# nohup python manage.py runserver 192.168.220.131:8000 &
[1] 54337
(env_1) [root@centostest myobject]# nohup: ignoring input and appending output to ‘nohup.out’


```

