---
layout: post
title: DevOps之Ansible2.5部署
categories: DevOps Ansible
tags: DevOps Ansible
---

* content
{:toc}


### 准备linux初始环境


```shell
[root@ansible ~]# systemctl stop firewalld
[root@ansible ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@ansible ~]# sed -i "s#SELINUX=enforcing#SELINUX=disabled#g" /etc/sysconfig/selinux
[root@ansible ~]# more /etc/sysconfig/selinux 

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

[root@ansible ~]# setenforce 0
[root@ansible ~]# getenforce
Permissive
[root@ansible ~]# yum -y install git nss curl wget

```

### Ansible2.5+Python3.6安装步骤

#### 安装python3.6.5和virtualenv工具

```shell
[root@ansible ~]# wget http://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
[root@ansible ~]# tar xf Python-3.6.5.tar.xz
[root@ansible ~]# cd Python-3.6.5
[root@ansible Python-3.6.5]# yum -y install gcc
[root@ansible Python-3.6.5]# ./configure --prefix=/usr/local/ --with-ensurepip=install --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"

[root@ansible Python-3.6.5]# yum install zlib*  -y
[root@ansible Python-3.6.5]# make && make altinstall

Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-9.0.3 setuptools-39.0.1

[root@ansible Python-3.6.5]# which pip3.6
/usr/local/bin/pip3.6
[root@ansible Python-3.6.5]# ln -s /usr/local/bin/pip3.6 /usr/local/bin/pip
[root@ansible Python-3.6.5]# which pip
/usr/local/bin/pip

[root@ansible Python-3.6.5]# yum install -y openssl*
[root@ansible Python-3.6.5]# ./configure --prefix=/usr/local/ --with-ensurepip=install --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib" --with-ssl
[root@ansible Python-3.6.5]# make && make altinstall
[root@ansible Python-3.6.5]# ln -s /usr/local/bin/pip3.6 /usr/local/bin/pip
ln: failed to create symbolic link ‘/usr/local/bin/pip’: File exists

[root@ansible Python-3.6.5]# pip install virtualenv
Collecting virtualenv
  Downloading https://files.pythonhosted.org/packages/eb/9f/33373d471bb9337c8db86b763052964c42f5079ea0de9517bc88acfbad26/virtualenv-16.6.2-py2.py3-none-any.whl (2.0MB)
    100% |████████████████████████████████| 2.0MB 309kB/s 
Installing collected packages: virtualenv
Successfully installed virtualenv-16.6.2
You are using pip version 9.0.3, however version 19.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.

```




#### 创建ansible账户并安装python3.6.5版本virtualenv实例

```shell
[root@ansible Python-3.6.5]# cd 
[root@ansible ~]# useradd deploy && su - deploy
[deploy@ansible ~]$ virtualenv -p /usr/local/bin/python3.6  .py3-a2.5-env
Already using interpreter /usr/local/bin/python3.6
Using base prefix '/usr/local'
New python executable in /home/deploy/.py3-a2.5-env/bin/python3.6
Also creating executable in /home/deploy/.py3-a2.5-env/bin/python
Installing setuptools, pip, wheel...
done.

```

#### Git源码安装ansible2.5


```shell
[deploy@ansible ~]$ cd /home/deploy/.py3-a2.5-env/
[deploy@ansible .py3-a2.5-env]$ git clone https://github.com/ansible/ansible.git
Cloning into 'ansible'...
remote: Enumerating objects: 137, done.
remote: Counting objects: 100% (137/137), done.
remote: Compressing objects: 100% (65/65), done.
remote: Total 452558 (delta 74), reused 89 (delta 60), pack-reused 452421
Receiving objects: 100% (452558/452558), 161.43 MiB | 253.00 KiB/s, done.
Resolving deltas: 100% (295204/295204), done.


```


#### 加载python3.6.5 virtualenv环境
```shell
[deploy@ansible .py3-a2.5-env]$ source /home/deploy/.py3-a2.5-env/bin/activate
(.py3-a2.5-env) [deploy@ansible .py3-a2.5-env]$

```

#### 安装ansible依赖包
```shell
(.py3-a2.5-env) [deploy@ansible .py3-a2.5-env]$ pip install paramiko PyYAML jinja2
(.py3-a2.5-env) [deploy@ansible .py3-a2.5-env]$ ll
total 8
drwxrwxr-x. 15 deploy deploy 4096 Jul 21 15:42 ansible
drwxrwxr-x.  2 deploy deploy 4096 Jul 20 18:50 bin
drwxrwxr-x.  2 deploy deploy   24 Jul 20 18:49 include
drwxrwxr-x.  3 deploy deploy   23 Jul 20 18:49 lib
(.py3-a2.5-env) [deploy@ansible .py3-a2.5-env]$ pwd
/home/deploy/.py3-a2.5-env




```

#### 在python3.6.5虚拟环境下加载ansible2.5
```shell

(.py3-a2.5-env) [deploy@ansible .py3-a2.5-env]$ cd ansible/
(.py3-a2.5-env) [deploy@ansible ansible]$ pwd
/home/deploy/.py3-a2.5-env/ansible

#将ansible切换到2.5版本
[deploy@ansible ansible]$ git checkout stable-2.5
Branch stable-2.5 set up to track remote branch stable-2.5 from origin.
Switched to a new branch 'stable-2.5'

#在此虚拟环境下加载ansible2.5版本
(.py3-a2.5-env) [deploy@ansible ansible]$ source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q

```

#### 验证ansible版本
```shell
(.py3-a2.5-env) [deploy@ansible ansible]$ ansible --version
ansible 2.5.15 (stable-2.5 ab16969416) last updated 2019/07/21 15:42:47 (GMT +800)
  config file = None
  configured module search path = ['/home/deploy/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/deploy/.py3-a2.5-env/ansible/lib/ansible
  executable location = /home/deploy/.py3-a2.5-env/ansible/bin/ansible
  python version = 3.6.5 (default, Jul 20 2019, 18:43:01) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
(.py3-a2.5-env) [deploy@ansible ansible]$ exit
logout
[root@ansible ~]#
```



### 测试验证

#### 开始编写playbooks

```shell
[root@ansible ~]# su - deploy
Last login: Sun Jul 21 15:39:53 CST 2019 on pts/0
[deploy@ansible ~]$ source .py3-a2.5-env/bin/activate
(.py3-a2.5-env) [deploy@ansible ~]$ source .py3-a2.5-env/ansible/hacking/env-setup -q
(.py3-a2.5-env) [deploy@ansible ~]$ ansible-playbook --version
ansible-playbook 2.5.15 (stable-2.5 ab16969416) last updated 2019/07/21 15:42:47 (GMT +800)
  config file = None
  configured module search path = ['/home/deploy/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/deploy/.py3-a2.5-env/ansible/lib/ansible
  executable location = /home/deploy/.py3-a2.5-env/ansible/bin/ansible-playbook
  python version = 3.6.5 (default, Jul 20 2019, 18:43:01) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]

(.py3-a2.5-env) [deploy@ansible ~]$ mkdir test-playbooks
(.py3-a2.5-env) [deploy@ansible ~]$ cd test-playbooks/
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ mkdir inventory
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ mkdir roles
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ cd inventory
(.py3-a2.5-env) [deploy@ansible inventory]$ vim testenv
[testservers]
test.example.com

[testservers:vars]
server_name=test.example.com
user=root
output=/root/test.txt

(.py3-a2.5-env) [deploy@ansible inventory]$ cd ..
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ls
inventory  roles
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ cd roles/
(.py3-a2.5-env) [deploy@ansible roles]$ mkdir testbox
(.py3-a2.5-env) [deploy@ansible roles]$ cd testbox/
(.py3-a2.5-env) [deploy@ansible testbox]$ mkdir tasks
(.py3-a2.5-env) [deploy@ansible testbox]$ cd tasks/
(.py3-a2.5-env) [deploy@ansible tasks]$ vim main.yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is loggging {{ server_name }}' > {{ output }}"
  
(.py3-a2.5-env) [deploy@ansible tasks]$ pwd
/home/deploy/test-playbooks/roles/testbox/tasks
(.py3-a2.5-env) [deploy@ansible tasks]$ cd ../../..
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ pwd
/home/deploy/test-playbooks
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ vim deploy.yml
- hosts: "testservers"
  gather_facts: true
  remote_user: root
  roles:
    - testbox

(.py3-a2.5-env) [deploy@ansible test-playbooks]$ tree .
.
├── deploy.yml
├── inventory
│   └── testenv
└── roles
    └── testbox
        └── tasks
            └── main.yml

4 directories, 3 files
```

#### 配置SSH免秘钥认证
```shell
[root@ansible ~]# vim /etc/hosts
192.168.8.223 test.example.com

(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/deploy/.ssh/id_rsa): 
Created directory '/home/deploy/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/deploy/.ssh/id_rsa.
Your public key has been saved in /home/deploy/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:/P8fYVLaAacdGE+NT/UfVMJfFoC3X2sgRsTEEIL3V4o deploy@ansible
The key''s randomart image is:
+---[RSA 2048]----+
|      .. oBo.==BB|
|     . ..  = +@.B|
|      . . o +..@o|
|       . E = o+ B|
|        S o .oo++|
|         .    o+.|
|          .   .. |
|           .    .|
|            .....|
+----[SHA256]-----+
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ssh-copy-id -i /home/deploy/.ssh/id_rsa.pub root@test.example.com
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/deploy/.ssh/id_rsa.pub"
The authenticity of host 'test.example.com (192.168.8.223)' can't be established.
ECDSA key fingerprint is SHA256:6V8EVnndfg2MH6GAguZ8zHf7FQekzy1ejUe5rdxiqI4.
ECDSA key fingerprint is MD5:df:2a:6b:89:0e:4e:8b:58:91:7f:7c:aa:bb:63:6b:ab.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@test.example.com's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@test.example.com'"
and check to make sure that only the key(s) you wanted were added.

(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ssh root@test.example.com
Last login: Sun Jul 21 05:18:28 2019 from 192.168.8.163
[root@testbox ~]# whoami
root
```
#### 验证
```shell
[root@testbox ~]# exit
logout
Connection to test.example.com closed.
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ls
deploy.yml  inventory  roles
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ansible-playbook -i inventory/testenv ./deploy.yml

PLAY [testservers] ************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
ok: [test.example.com]

TASK [testbox : Print server name and user to remote testbox] *****************************************************************
changed: [test.example.com]

PLAY RECAP ********************************************************************************************************************
test.example.com           : ok=2    changed=1    unreachable=0    failed=0  

#以下内容可以看出已经成功在远程被部署主机test.example.com上创建一个test.txt文件，且文件内容与预先设置的一致
(.py3-a2.5-env) [deploy@ansible test-playbooks]$ ssh root@test.example.com
Last login: Sun Jul 21 05:27:32 2019 from 192.168.8.222
[root@testbox ~]# ls
anaconda-ks.cfg  test.txt
[root@testbox ~]# cat test.txt
Currently root is loggging test.example.com


```

