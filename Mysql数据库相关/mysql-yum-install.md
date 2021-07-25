---
title: Mysql 5.7 使用 yum安装
permalink: mysql-yum-install
date: 2020-05-04 00:46:26
updated: 2020-05-04 19:59:49
tags: 
    - mysql
    - 数据库
categories: 数据库
toc: true
excerpt: 在CentOS7中默认安装有MariaDB，这个是MySQL的分支，但为了需要，还是要在系统中安装MySQL，而且安装完成之后可以直接覆盖掉MariaDB。
---

在CentOS7中默认安装有MariaDB，这个是MySQL的分支，但为了需要，还是要在系统中安装MySQL，而且安装完成之后可以直接覆盖掉MariaDB。

### 1.安装wget工具

使用wget工具下载mysql安装包和yum源文件

```shell
[root@node02 ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
-bash: wget: command not found
You have new mail in /var/spool/mail/root
[root@node02 ~]# yum install -y wget
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.jdcloud.com
 * extras: mirror.jdcloud.com
 * updates: mirrors.tuna.tsinghua.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package wget.x86_64 0:1.14-18.el7_6.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================
 Package            Arch                 Version                          Repository          Size
===================================================================================================
Installing:
 wget               x86_64               1.14-18.el7_6.1                  base               547 k

Transaction Summary
===================================================================================================
Install  1 Package

Total download size: 547 k
Installed size: 2.0 M
Downloading packages:
wget-1.14-18.el7_6.1.x86_64.rpm                                             | 547 kB  00:00:08     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : wget-1.14-18.el7_6.1.x86_64                                                     1/1 
  Verifying  : wget-1.14-18.el7_6.1.x86_64                                                     1/1 

Installed:
  wget.x86_64 0:1.14-18.el7_6.1                                                                    

Complete!
[root@node02 ~]# 
```

### 2. 下载并安装MySQL官方的 Yum Repository

```shell
[root@node02 ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
--2019-10-11 14:31:09--  http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
Resolving dev.mysql.com (dev.mysql.com)... 137.254.60.11
Connecting to dev.mysql.com (dev.mysql.com)|137.254.60.11|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm [following]
--2019-10-11 14:31:11--  https://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
Connecting to dev.mysql.com (dev.mysql.com)|137.254.60.11|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://repo.mysql.com//mysql57-community-release-el7-10.noarch.rpm [following]
--2019-10-11 14:31:14--  https://repo.mysql.com//mysql57-community-release-el7-10.noarch.rpm
Resolving repo.mysql.com (repo.mysql.com)... 104.93.1.42
Connecting to repo.mysql.com (repo.mysql.com)|104.93.1.42|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 25548 (25K) [application/x-redhat-package-manager]
Saving to: ‘mysql57-community-release-el7-10.noarch.rpm’

100%[=========================================================>] 25,548      --.-K/s   in 0.001s  

2019-10-11 14:31:15 (31.3 MB/s) - ‘mysql57-community-release-el7-10.noarch.rpm’ saved [25548/25548]

-c: No such file or directory
No URLs found in -c.
FINISHED --2019-10-11 14:31:15--
Total wall clock time: 5.9s
Downloaded: 1 files, 25K in 0.001s (31.3 MB/s)
You have new mail in /var/spool/mail/root
[root@node02 ~]# 
```

使用上面的命令就直接下载了安装用的Yum Repository，大概25KB的样子，然后就可以直接yum安装了。

```shell
[root@node02 ~]#  yum -y install mysql57-community-release-el7-10.noarch.rpm 
Loaded plugins: fastestmirror
Examining mysql57-community-release-el7-10.noarch.rpm: mysql57-community-release-el7-10.noarch
Marking mysql57-community-release-el7-10.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package mysql57-community-release.noarch 0:el7-10 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================
 Package                     Arch     Version     Repository                                  Size
===================================================================================================
Installing:
 mysql57-community-release   noarch   el7-10      /mysql57-community-release-el7-10.noarch    30 k

Transaction Summary
===================================================================================================
Install  1 Package

Total size: 30 k
Installed size: 30 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : mysql57-community-release-el7-10.noarch                                         1/1 
  Verifying  : mysql57-community-release-el7-10.noarch                                         1/1 

Installed:
  mysql57-community-release.noarch 0:el7-10                                                        

Complete!
You have new mail in /var/spool/mail/root
[root@node02 ~]# 
```

下面就是使用yum安装MySQL了,这步可能会花些时间，安装完成后就会覆盖掉之前的mariadb。

```shell
[root@node02 ~]# yum -y install mysql-community-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.jdcloud.com
 * extras: mirror.jdcloud.com
 * updates: mirrors.tuna.tsinghua.edu.cn
mysql-connectors-community                                                  | 2.5 kB  00:00:00     
mysql-tools-community                                                       | 2.5 kB  00:00:00     
mysql57-community                                                           | 2.5 kB  00:00:00     
(1/3): mysql57-community/x86_64/primary_db                                  | 184 kB  00:00:01     
(2/3): mysql-tools-community/x86_64/primary_db                              |  61 kB  00:00:02     
(3/3): mysql-connectors-community/x86_64/primary_db                         |  44 kB  00:00:03     
Resolving Dependencies
--> Running transaction check
---> Package mysql-community-server.x86_64 0:5.7.27-1.el7 will be installed
--> Processing Dependency: mysql-community-common(x86-64) = 5.7.27-1.el7 for package: mysql-community-server-5.7.27-1.el7.x86_64
--> Processing Dependency: mysql-community-client(x86-64) >= 5.7.9 for package: mysql-community-server-5.7.27-1.el7.x86_64
--> Running transaction check
---> Package mysql-community-client.x86_64 0:5.7.27-1.el7 will be installed
--> Processing Dependency: mysql-community-libs(x86-64) >= 5.7.9 for package: mysql-community-client-5.7.27-1.el7.x86_64
---> Package mysql-community-common.x86_64 0:5.7.27-1.el7 will be installed
--> Running transaction check
---> Package mariadb-libs.x86_64 1:5.5.60-1.el7_5 will be obsoleted
--> Processing Dependency: libmysqlclient.so.18()(64bit) for package: 2:postfix-2.10.1-7.el7.x86_64
--> Processing Dependency: libmysqlclient.so.18(libmysqlclient_18)(64bit) for package: 2:postfix-2.10.1-7.el7.x86_64
---> Package mysql-community-libs.x86_64 0:5.7.27-1.el7 will be obsoleting
--> Running transaction check
---> Package mysql-community-libs-compat.x86_64 0:5.7.27-1.el7 will be obsoleting
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================
 Package                           Arch         Version              Repository               Size
===================================================================================================
Installing:
 mysql-community-libs              x86_64       5.7.27-1.el7         mysql57-community       2.2 M
     replacing  mariadb-libs.x86_64 1:5.5.60-1.el7_5
 mysql-community-libs-compat       x86_64       5.7.27-1.el7         mysql57-community       2.0 M
     replacing  mariadb-libs.x86_64 1:5.5.60-1.el7_5
 mysql-community-server            x86_64       5.7.27-1.el7         mysql57-community       165 M
Installing for dependencies:
 mysql-community-client            x86_64       5.7.27-1.el7         mysql57-community        24 M
 mysql-community-common            x86_64       5.7.27-1.el7         mysql57-community       275 k

Transaction Summary
===================================================================================================
Install  3 Packages (+2 Dependent packages)

Total download size: 194 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/mysql57-community/packages/mysql-community-common-5.7.27-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Public key for mysql-community-common-5.7.27-1.el7.x86_64.rpm is not installed
(1/5): mysql-community-common-5.7.27-1.el7.x86_64.rpm                       | 275 kB  00:00:01     
(2/5): mysql-community-libs-5.7.27-1.el7.x86_64.rpm                         | 2.2 MB  00:00:04     
(3/5): mysql-community-libs-compat-5.7.27-1.el7.x86_64.rpm                  | 2.0 MB  00:00:06     
mysql-community-client-5.7.27- FAILED                                          18 MB 170:42:08 ETA 
http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql-community-client-5.7.27-1.el7.x86_64.rpm: [Errno 12] Timeout on http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql-community-client-5.7.27-1.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
mysql-community-server-5.7.27- FAILED                                          16 MB  --:--:-- ETA 
http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql-community-server-5.7.27-1.el7.x86_64.rpm: [Errno 12] Timeout on http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql-community-server-5.7.27-1.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(4/5): mysql-community-client-5.7.27-1.el7.x86_64.rpm                       |  24 MB  00:11:58     
mysql-community-server-5.7.27- FAILED                                          4 MB 2600:47:50 ETA 
http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql-community-server-5.7.27-1.el7.x86_64.rpm: [Errno 12] Timeout on http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql-community-server-5.7.27-1.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(5/5): mysql-community-server-5.7.27-1.el7.x86_64.rpm                       | 165 MB  00:07:44     
---------------------------------------------------------------------------------------------------
Total                                                              148 kB/s | 194 MB  00:22:20     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Importing GPG key 0x5072E1F5:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: a4a9 4068 76fc bd3c 4567 70c8 8c71 8d3b 5072 e1f5
 Package    : mysql57-community-release-el7-10.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : mysql-community-common-5.7.27-1.el7.x86_64                                      1/6 
  Installing : mysql-community-libs-5.7.27-1.el7.x86_64                                        2/6 
  Installing : mysql-community-client-5.7.27-1.el7.x86_64                                      3/6 
  Installing : mysql-community-server-5.7.27-1.el7.x86_64                                      4/6 
  Installing : mysql-community-libs-compat-5.7.27-1.el7.x86_64                                 5/6 
  Erasing    : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                                            6/6 
  Verifying  : mysql-community-libs-compat-5.7.27-1.el7.x86_64                                 1/6 
  Verifying  : mysql-community-common-5.7.27-1.el7.x86_64                                      2/6 
  Verifying  : mysql-community-server-5.7.27-1.el7.x86_64                                      3/6 
  Verifying  : mysql-community-client-5.7.27-1.el7.x86_64                                      4/6 
  Verifying  : mysql-community-libs-5.7.27-1.el7.x86_64                                        5/6 
  Verifying  : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                                            6/6 

Installed:
  mysql-community-libs.x86_64 0:5.7.27-1.el7    mysql-community-libs-compat.x86_64 0:5.7.27-1.el7 
  mysql-community-server.x86_64 0:5.7.27-1.el7 

Dependency Installed:
  mysql-community-client.x86_64 0:5.7.27-1.el7     mysql-community-common.x86_64 0:5.7.27-1.el7    

Replaced:
  mariadb-libs.x86_64 1:5.5.60-1.el7_5                                                             
#提示安装完成,表示安装成功
Complete!
You have new mail in /var/spool/mail/root

#已经查询不到mariadb数据库了
[root@node02 ~]# rpm -qa|grep mariadb
You have new mail in /var/spool/mail/root
[root@node02 ~]# 
```

## 3. MySQL数据库设置

首先启动MySQL

```shell
#启动mysql服务
[root@node02 ~]# systemctl start  mysqld.service
#查看mysql运行状态
[root@node02 ~]# systemctl status  mysqld.service  
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   #表示已经启动(linux)
   Active: active (running) since Fri 2019-10-11 15:14:57 CST; 6s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 22525 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 22449 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 22528 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─22528 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Oct 11 15:14:54 node02.kaikeba.com systemd[1]: Starting MySQL Server...
Oct 11 15:14:57 node02.kaikeba.com systemd[1]: Started MySQL Server.
[root@node02 ~]# 
```

此时MySQL已经开始正常运行，不过要想进入MySQL还得先找出此时root用户的密码，通过如下命令可以在日志文件中找出密码：

```shell
#查找到root用户登录mysql数据库的密码:7UOv>SVzygyB
[root@node02 ~]# grep "password" /var/log/mysqld.log
2019-10-11T07:14:54.482816Z 1 [Note] A temporary password is generated for root@localhost: 7UOv>SVzygyB
You have new mail in /var/spool/mail/root
[root@node02 ~]# 
```

命令进入数据库：

```shell
[root@node02 ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.27

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
#提示修改初始密码
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
#注意密码设置不能过于简单，mysql有密码设置规范（特殊字符、字母大小写，数字三者的组合）
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '!Qaz123456';
#添加scm用户对scm库的访问权限
mysql> grant all on scm.* to scm@'%' identified by '!Qaz123456';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> select user,host from user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| scm           | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
4 rows in set (0.01 sec)

#刷新访问权限的设置,这一步非常重要，如果没有操作，scm远程访问mysql数据库就失败.
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

#添加root用户远程访问数据库
mysql>grant all on *.* to root@'%' identified by '!Qaz123456';
mysql> flush privileges;
mysql> select user,host from user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| scm           | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
5 rows in set (0.00 sec)

mysql> delete from user where user='root' and host='localhost';
Query OK, 1 row affected (0.02 sec)

mysql> select user,host from user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| scm           | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
+---------------+-----------+
4 rows in set (0.00 sec)

mysql> flush privileges;

#创建scm数据库
mysql> create database scm;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| scm                |
| sys                |
+--------------------+
5 rows in set (0.00 sec)


mysql> update mysql.user set Grant_priv='Y',Super_priv='Y' where user = 'root' and host = '%';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> quit
Bye
You have new mail in /var/spool/mail/root
#从起mysql服务
[root@node02 ~]# systemctl restart mysqld.service
[root@node02 ~]# 
```
