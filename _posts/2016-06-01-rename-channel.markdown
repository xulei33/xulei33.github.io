---
layout:     post
title:      "CI6.3 Linux下连接DB2"
subtitle:   "SAS/Access DB2"
date:       2016-05-16 21:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - CI6x
    - Linux
---

### SAS计算层服务器安装DB2客户端

![](/img/in-post/2016-05-17/2016-05-17(0).png)
![](/img/in-post/2016-05-17/2016-05-17(1).png)
![](/img/in-post/2016-05-17/2016-05-17(2).png)
![](/img/in-post/2016-05-17/2016-05-17(3).png)
![](/img/in-post/2016-05-17/2016-05-17(4).png)
![](/img/in-post/2016-05-17/2016-05-17(5).png)
![](/img/in-post/2016-05-17/2016-05-17(6).png)
![](/img/in-post/2016-05-17/2016-05-17(7).png)
![](/img/in-post/2016-05-17/2016-05-17(8).png)
![](/img/in-post/2016-05-17/2016-05-17(9).png)
![](/img/in-post/2016-05-17/2016-05-17(10).png)
![](/img/in-post/2016-05-17/2016-05-17(11).png)



### 添加DB2环境变量
> vi /etc/profile
>
> DB2_HOME=/opt/ibm/db2/v10.5
>
> PATH=$DB2_HOME/bin:$PATH
>
> export DB2_HOME
>
> export PATH
> 
> source /home/db2inst1/sqllib/db2profile
> 
> export DB2CODPEAGE=1386
>

![](/img/in-post/2016-05-17/2016-05-17(12).png)
![](/img/in-post/2016-05-17/2016-05-17(13).png)

### 添加DB2环境变量到SAS
> vi SASHome/SASFoundation/9.4/bin/sasenv_local
>
> . /home/db2inst1/sqllib/db2profile
>

![](/img/in-post/2016-05-17/2016-05-17(14).png)
![](/img/in-post/2016-05-17/2016-05-17(15).png)

### 配置SAS Format
> vi SASConfig/Lev1/SASApp/appserver_autoexec_usermods.sas
>
> options FMTSEARCH=(MAMISC WORK LIBRARY) NOFMTERR;
>
> options missing="";
>

### 配置MALauncher支持中文

> vi /SASHome/SASMarketingAutomationLauncher/6.3/sasmalauncher.ini
>
> JavaArgs_8=-Dma.unicode.marker=U+
> JavaArgs_9=-Dma.launcher.log=malog.log
> JavaArgs_10=-Dma.launcher.abslogdir=/zhyx/SASConfig/Lev1/Logs
>

### 配置MOLauncher支持中文

> vi /SASHome/SASMarketingOptimizationLauncher/6.3/sasmolauncher.ini
>
> JavaArgs_8=-Dma.unicode.marker=U+
> JavaArgs_9=-Dma.launcher.log=malog.log
> JavaArgs_10=-Dma.launcher.abslogdir=/zhyx/SASConfig/Lev1/Logs
>

> /SASHome/SASMarketingAutomationLauncher/6.3/sasmalauncher.ini
>
> JavaArgs_8=-Dma.unicode.marker=U+
> JavaArgs_9=-Dma.launcher.log=malog.log
> JavaArgs_10=-Dma.launcher.abslogdir=/zhyx/SASConfig/Lev1/Logs
>


### 配置sasv9.cfg
> vi SASHome\SASFoundation\9.4\sasv9.cfg

#### 配置SAS支持中文 
sasv9.cfg文件尾部添加
> -VALIDVARNAME ANY
> -VALIDMEMNAME EXTEND
> -DBCS
> -DBCSTYPE PCMS
> -DBCSLANG CHINESE

#### 配置-JREOPTIONS
增加下面两个jre参数
> -Duser.language=zh
>
> -Duser.country=CN

#### 配置计算服务器-WORK工作目录

> 在计算服务器共享存储上创建计算服务器工作目录
> mkdir -p /compute/saswork
> chown 775 /compute/saswork

>
> -WORK /compute/saswork
> 

### 计算层服务器编目

>
> cd /opt/ibm/db2/V10.5/bin
>
> ./db2 catalog tcpip node sasdb remote DB2Server server 60000
>
> ./db2 catalog database sasdb at node sasdb
>
> ./db2 connect to sasdb user db2inst1 using sas123
>

### SMC创建代理用户组，认证域DBAuth

![](/img/in-post/2016-05-17/2016-05-17(16).png)
![](/img/in-post/2016-05-17/2016-05-17(17).png)
![](/img/in-post/2016-05-17/2016-05-17(18).png)

### SMC创建DB2Server

![](/img/in-post/2016-05-17/2016-05-17(19).png)
![](/img/in-post/2016-05-17/2016-05-17(20).png)
![](/img/in-post/2016-05-17/2016-05-17(21).png)
![](/img/in-post/2016-05-17/2016-05-17(22).png)
![](/img/in-post/2016-05-17/2016-05-17(23).png)


### SAS EG创建CDM表和序列

CDM的数据库初始化脚本保存在SASHome/SASFoundation/9.4/misc/cicsvr目录下面，需要通过EG执行下面两个sas程序完成CDM创建。

> 1. ci_cdm_ddl_db2.sas 
> 2. ci_cdm_load_codes.sas
>

### SMC创建数据集
![](/img/in-post/2016-05-17/2016-05-17(24).png)
![](/img/in-post/2016-05-17/2016-05-17(25).png)
![](/img/in-post/2016-05-17/2016-05-17(26).png)
![](/img/in-post/2016-05-17/2016-05-17(27).png)
![](/img/in-post/2016-05-17/2016-05-17(28).png)
![](/img/in-post/2016-05-17/2016-05-17(29).png)


### 配置EG代码目录
![](/img/in-post/2016-05-17/2016-05-17(30).png)
![](/img/in-post/2016-05-17/2016-05-17(31).png)
![](/img/in-post/2016-05-17/2016-05-17(32).png)


