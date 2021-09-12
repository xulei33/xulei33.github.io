---
layout:     post
title:      "SAS ODBC连接MySQL"
subtitle:   "Windows Server 2012"
date:       2017-05-09 23:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - MySQL
    - SAS
---

> 通过SAS Access ODBC接口，SAS可以连接处理很多数据库数据。

## 安装MySQL

安装MySQL的方法见[Windows Install MySQL 5.7.18](http://xulei.org/2017/05/09/windows-mysql/)

## 安装MySQL ODBC驱动
通过地址[https://dev.mysql.com/downloads/connector/odbc/](https://dev.mysql.com/downloads/connector/odbc/)下载MySQL的ODBC驱动

```
mysql-connector-odbc-5.3.8-winx64.msi
```

通过系统管理员Administrator登录系统安装。

## 配置ODBC源

![](/img/in-post/2017-05-09/12.png)
![](/img/in-post/2017-05-09/13.png)
![](/img/in-post/2017-05-09/14.png)
![](/img/in-post/2017-05-09/15.png)


## 配置SAS数据库代理用户组

通过SAS管理员登录SMC，创建代理用户组和认证域。

![](/img/in-post/2017-05-09/16.png)
![](/img/in-post/2017-05-09/17.png)
![](/img/in-post/2017-05-09/18.png)


## 注册ODBC逻辑库

通过SAS管理员登录SMC，注册ODBC逻辑库
![](/img/in-post/2017-05-09/19.png)
![](/img/in-post/2017-05-09/20.png)
![](/img/in-post/2017-05-09/21.png)
![](/img/in-post/2017-05-09/22.png)
![](/img/in-post/2017-05-09/23.png)

## 为逻辑库注册表
![](/img/in-post/2017-05-09/24.png)
![](/img/in-post/2017-05-09/25.png)

## 通过EG验证逻辑配置
![](/img/in-post/2017-05-09/26.png)


