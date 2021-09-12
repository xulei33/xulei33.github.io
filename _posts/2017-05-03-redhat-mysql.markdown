---
layout:     post
title:      "RedHat 7.2 Install MySQL 5.7.18"
subtitle:   "rpm install MySQL"
date:       2017-05-03 21:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - MySQL
    - Linux
---

> 
Oracle提供了yum源和rpm包的方式安装MySQL，yum源方式比较简单，通过命令yum install mysql-community-{server,client,common,libs}-* 既可完成安装。
本文记录更常用的通过在Oracle官网下载RPM包的方式手工安装MySQL。

## 卸载mariadb 
Redhat系统默认安装了mariadb，在安装MySQL之前需要先卸载掉。

```
rpm -qa|grep mariadb
rpm -qa|grep mariadb|xargs rpm -e --nodeps
```

## 卸载MySQL
检查服务器上是否已经安装了MySQL，有的话先卸载掉。

```
rpm -qa|grep MySQL
rpm -qa|grep MySQL|xargs rpm -e --nodeps
rpm -qa|grep mysql
rpm -qa|grep mysql|xargs rpm -e --nodeps
```

## Download MySQL rpm

https://dev.mysql.com/downloads/mysql/5.5.html#downloads
按下图选择操作系统类型和版本
![](/img/in-post/2017-05-03/download-mysql.png)

下载下面几个rpm软件包

```
1. mysql-community-server-5.7.18-1.el7.x86_64.rpm
2. mysql-community-client-5.7.18-1.el7.x86_64.rpm
3. mysql-community-devel-5.7.18-1.el7.x86_64.rpm
4. mysql-community-common-5.7.18-1.el7.x86_64.rpm
5. mysql-community-libs-5.7.18-1.el7.x86_64.rpm
```

## root 安装MySQL

```
rpm -ivh mysql-community-common-5.7.18-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.18-1.el7.x86_64.rpm
rpm -ivh mysql-community-devel-5.7.18-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.18-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.18-1.el7.x86_64.rpm

```

## MySQL文件目录
![](/img/in-post/2017-05-03/mysql-files.png)

## 修改配置文件，跳过密码检查
vi /etc/my.cnf
加入 skip-grant-tables
跳过密码检测

## 启动MySQL

```
service mysqld start 启动mysql
mysql -uroot -p 登录，无需密码直接回车
mysql>use mysql; 选择数据库
mysql>update user set password_expired='N' where user='root';
设置密码失效为”N”
mysql> update user set authentication_string=password('sas123') where user='root';
设置密码为”sas123”
mysql> flush privileges;
刷新生效
mysql>quit;
退出
```

## 停启MySQL

```
systemctl stop mysqld
systemctl start mysqld
systemctl restart mysqld
```

## 修改配置

```
vi /etc/my.cnf
注释或删除 #skip-grant-tables
mysql -u root -p
用新密码sas123登陆
```

## 修改密码

```
[root@mysql MySQL]#mysqladmin -u用户名 -p旧密码 password 新密码
[root@mysql MySQL]#mysqladmin -uroot –p123456 password 'sas123'
mysql>set password=password('sas123'); 
```

## 允许mysql远程访问

```
赋予root任何主机访问的权限
mysql>grant all privileges on *.* to ‘root’@’%’with grant option;
mysql>grant all privileges on *.* to ‘root’@’%’identified by ‘sas123’ with grant option;
```

## 配置其他参数

```
vi /etc/my.cnf

character-set-server=utf8
collation-server=utf8_general_ci
max_connections=1000
table_open_cache=6000
thread_cache_size=50
open_files_limit=8000
event_scheduler=ON
 
group_concat_max_len=999999999999
```

## 防火墙

```
查看防火墙状态
systemctl status firewalld
临时关闭防火墙命令。重启电脑后，防火墙自动起来
systemctl stop firewalld
永久关闭防火墙命令。重启后，防火墙不会自动启动
systemctl disable firewalld
打开防火墙命令systemctl enable firewalld
```

## MySQL需要关注的6个文件如下:

```
[root@mysql opt]# vi /etc/init.d/mysqld
[root@mysql opt]# vi /etc/my.cnf
[root@mysql opt]# vi /run/systemd/generator.late/mysqld.service
[root@mysql opt]# vi /run/systemd/generator.late/runlevel5.target.wants/mysqld.service
[root@mysql opt]# vi /run/systemd/generator.late/runlevel4.target.wants/mysqld.service
[root@mysql opt]# vi /run/systemd/generator.late/runlevel3.target.wants/mysqld.service
```


