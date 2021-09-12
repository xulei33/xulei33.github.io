---
layout:     post
title:      "Trojan SwitchyOmega手工安装配置"
subtitle:   "Trojan SwitchyOmega安装配置说明"
date:       2021-09-11 12:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Trojan
    - SwitchyOmega
---

> msi版本mysql双击文件即可安装，相对简单，本文不介绍此版本安装。

## 下载安装包

通过 [https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/) 下载MySQL安装包，选择版本：Windows (x86, 64-bit), ZIP Archive。
下载完后，解压到系统目录，如：

```
C:\MySQL\mysql-5.7.18
```


## 配置环境变量
打开环境变量配置页面(winserver服务器环境变量位置：服务器管理器->本地服务器->计算机名称->高级->环境变量)，在系统变量path后面添加mysql bin文件路径，例如：

```
C:\MySQL\mysql-5.7.18\bin
```

## 配置mysql
复制根目录下my-default.ini为my.ini,修改my.ini配置文件如下：

```
basedir = C:\MySQL\mysql-5.7.18
datadir = C:\MySQL\data(mysql数据库存放目录)
port = 3306(mysql对外开放端口，默认3306，可修改)
```

## 初始化mysql
使用系统管理员Administrator进入mysql的bin目录，执行数据库初始化命令，并安装为系统服务。

```
cd C:\MySQL\mysql-5.7.18\bin
mysqld --initialize-insecure
mysqld -install
```

使用快捷键win+r打开运行，执行services.msc查看服务，看看mysql服务是否安装成功。

## 启动、停止MySQL命令

```
net start mysql 启动MySQL
net stop mysql 停止MySQL
```

## 登录用户管理及密码修改

```
mysql -uroot -p 登录，无需密码直接回车
mysql>use mysql; 选择数据库
mysql>update user set password_expired='N' where user='root'; 设置密码失效为”N”

修改密码为”sas123”
mysql> update user set authentication_string=password('sas123') where user='root';
mysql> flush privileges;刷新生效
mysql>quit;退出
```

## 允许mysql远程访问

```
赋予root任何主机访问的权限
mysql>grant all privileges on *.* to ‘root’@’%’ with grant option;
```

## 新建允许远程链接mysql数据库的用户

```
grant all on *.* to sas@'%' identified by 'sas123' with grant option;
flush privileges;
```

创建一个登录名为sas，密码为sas123供任意ip访问的用户(%可用具体ip替代)

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
