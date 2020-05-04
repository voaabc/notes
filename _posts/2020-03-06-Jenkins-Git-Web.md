---
layout: post
title: Jenkins Git Web
categories: Jenkins Git
tags: Jenkins Git
---

* content
{:toc}



### 001-Git
```sh
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
[root@localhost ~]# yum install git -y

[root@localhost ~]# iptables -F
[root@localhost ~]# useradd git
[root@localhost ~]# passwd git
Changing password for user git.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@localhost ~]# su - git
[git@localhost ~]$ pwd
/home/git
[git@localhost ~]$ mkdir repos
[git@localhost ~]$ cd repos/
[git@localhost repos]$ mkdir app.git
[git@localhost repos]$ cd app.git/
[git@localhost app.git]$ ls
[git@localhost app.git]$ ls
[git@localhost app.git]$ git --bare init
Initialized empty Git repository in /home/git/repos/app.git/
[git@localhost app.git]$ ls -a
.  ..  branches  config  description  HEAD  hooks  info  objects  refs
```

### 002-Web
```sh
[root@localhost ~]# yum install git -y
[root@localhost ~]# mkdir test
[root@localhost ~]# cd test/
[root@localhost test]# git clone git@192.168.8.62:/home/git/repos/app.git

[root@localhost test]# ls
app
[root@localhost test]#  cd app/
[root@localhost app]# ls
[root@localhost app]# pwd
/root/test/app
[root@localhost app]# touch index.html

[root@localhost app]# 
[root@localhost app]# vi index.html 
[root@localhost app]# ll
total 4
-rw-r--r--. 1 root root 4 Mar  6 03:12 index.html
[root@localhost app]# git add .
[root@localhost app]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#   new file:   index.html
#
[root@localhost app]# git commit -m "1"
[master (root-commit) 2f16a70] 1
 Committer: root <root@localhost.localdomain>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly:

    git config --global user.name "Your Name"
    git config --global user.email you@example.com

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 1 insertion(+)
 create mode 100644 index.html
[root@localhost app]# git config --global user.name "Your Name"


[root@localhost app]# git status
# On branch master
nothing to commit, working directory clean
[root@localhost app]# git push origin master
git@192.168.8.62 s password: 
Counting objects: 3, done.
Writing objects: 100% (3/3), 208 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@192.168.8.62:/home/git/repos/app.git
 * [new branch]      master -> master

```


### 003-Web
```sh
[root@localhost ~]# mkdir test2
[root@localhost ~]# cd test
[root@localhost test]# cd ..
[root@localhost ~]# cd test2
[root@localhost test2]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:kIR51lEWS0s5xFzAN0a1Oa2pCOMQOuwcSUUA3qpMtnI root@localhost.localdomain
The key s randomart image is:
+---[RSA 2048]----+
|  ...=+..BO*o..  |
| . .ooo..+*o+  + |
|  . oo+   o+ .+ .|
|   + o o       + |
| o. * . S     o  |
|+..o o o o . .   |
|ooE o   . . .    |
|..               |
|                 |
+----[SHA256]-----+
[root@localhost test2]# cat ~/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2xeyhw4voa6vR0thVRUaYwR2QiRJUTyD/bFK7yD445C8wEZHXI+3QdqtUgWus
EyqfnsuXtxdHFgcTM8F95nLRXqz6ivIl6s4laUnARmsRhpXpxNrU62oJjvhFgKPVyjdxNUx7Mwl1rk8ZQeNKfQmOLq60aw+HNsPQ
pR7t2HnnqZRJW2H3sLtiCFQlbS8dJLJCNF0ItAtfPVAsFwhrE5tJXgLUg1ptyF0i04k1EMJCodqWAS6lrcIGs3SRTvHkiyPDLasr
o2lkSJh8oR9gVdbMH/Ge5mALJOyU+ySYX8zWUYDgT7hdv8L0YbCQ6PW+i/Hlc0m3/9A8rwfoRlaMUYBj root@localhost.localdomain

[root@localhost test2]# ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 git@192.168.8.62
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
git@192.168.8.62 s password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh -p '22' 'git@192.168.8.62'"
and check to make sure that only the key(s) you wanted were added.

[root@localhost ~]# ssh git@192.168.8.62
Last login: Fri Mar  6 03:40:15 2020
[git@localhost ~]$ exit
logout
Connection to 192.168.8.62 closed.
[root@localhost ~]#
```

### 004-Git

```sh
[git@localhost ~]$ cd .ssh/
[git@localhost .ssh]$ ll
total 4
-rw-------. 1 git git 408 Mar  6 04:31 authorized_keys
[git@localhost .ssh]$ more authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2xeyhw4voa6vR0thVRUaYwR2QiRJUTyD/bFK7yD445C8wEZHXI+3QdqtUgWus
EyqfnsuXtxdHFgcTM8F95nLRXqz6ivIl6s4laUnARmsRhpXpxNrU62oJjvhFgKPVyjdxNUx7Mwl1rk8ZQeNKfQmOLq60aw+HNsPQ
pR7t2HnnqZRJW2H3sLtiCFQlbS8dJLJCNF0ItAtfPVAsFwhrE5tJXgLUg1ptyF0i04k1EMJCodqWAS6lrcIGs3SRTvHkiyPDLasr
o2lkSJh8oR9gVdbMH/Ge5mALJOyU+ySYX8zWUYDgT7hdv8L0YbCQ6PW+i/Hlc0m3/9A8rwfoRlaMUYBj root@localhost.localdomain


```


### 005-Web
```sh
[root@localhost test2]# git clone git@192.168.8.62:/home/git/repos/app.git
Cloning into 'app'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.
[root@localhost test2]# ll app/
total 4
-rw-r--r--. 1 root root 4 Mar  6 04:35 index.html
```

### 006-Jenkins

```sh
[root@localhost ~]# sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
[root@localhost ~]# sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
[root@localhost ~]# yum install jenkins -y
[root@localhost ~]# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins
[root@localhost ~]# yum install java -y
[root@localhost ~]# java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (build 25.242-b08, mixed mode)
[root@localhost ~]# systemctl start jenkins
[root@localhost ~]# ps -ef | grep jenkins
jenkins   17313      1 77 08:17 ?        00:00:10 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized 
-Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war 
--logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 
--debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
root      17363  16947  0 08:17 pts/0    00:00:00 grep --color=auto jenkins
[root@localhost ~]# yum install net-tools -y
[root@localhost ~]# netstat -ntlp | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      17313/java

http://192.168.8.61:8080/

[root@localhost ~]# cd /var/lib/jenkins/updates/

[root@localhost updates]# sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

[root@localhost ~]# vim /var/lib/jenkins/hudson.model.UpdateCenter.xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
~        

[root@localhost ~]# systemctl stop jenkins
[root@localhost ~]# systemctl start jenkins

[root@localhost ~]# more /var/lib/jenkins/secrets/initialAdminPassword
009a7fdc898f4362a61f38c7b3000dcb
[root@localhost ~]# tail -f /var/log/jenkins/jenkins.log
```



```sh



```



```sh



```