---
layout:     post
title:      "Linux集群环境下一键停启SAS服务"
subtitle:   "1Key Star|Stop SAS Servers in Linux Cluster Env. "
date:       2016-04-16 17:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - SAS
    - Linux
---

# 背景

多层部署架构下的SAS软件通常会部署在多台服务器上，如元数据服务器至少会部署在3台服务器上，计算层服务器也会根据客户需要分析计算的数据量规模不同部署在几台到几十台服务器上，WebServer和WebAPPServer会参考系统使用人员数量至少部署在两台服务器上。另外SAS软件对系统启停还有一些特别要求，以上这些给SAS系统的服务器运维人员带来了比较大的麻烦。

>
> * 启动SAS服务器，需要使用用户`sas`执行`SASConfig/Lev1/sas.servers start`命令来启动SAS Server。需要按照`元数据主节点、元数据Node节点、计算层主节点、计算层Node节点、WebServer、中间层主节点、中间层Node节点服务器`的顺序启动SAS Server，其中计算层Node节点服务器可以并发启停，其他服务器需要用户等前一台服务器完成启动后，才能再启动下一台服务器。
>
> * 停止SAS服务的步骤同启动相反,需要使用系统用户`sas`执行`SASConfig/Lev1/sas.servers stop`命令来停止SAS Server。 中间层Node节点服务器和计算层Node节点服务器可以并发停止，其他服务器要等前一台停止完成后才能停止下一台。

在Linux环境下，通过配置服务器间的ssh互信，可以达到通过shell在一台服务器上执行其他服务器上shell的功能，这样我们可以通过编写一个shell脚本通过ssh命令来完成SAS多台服务器的启停工作，然后在通过配置开机启动和定时重启等，可以大大减少系统运维人员的工作量。

## 一键停启SAS服务样例说明
假设需要配置一键启停的SAS部署环境包含以下Redhat6.4服务器：

1. 三台元数据服务器:`sasmeta01、sasmeta02、sasmeta03`，其中`sasmeta01`为元数据主节点服务器，`sasmeta02`和`sasmeta03`为元数据Node节点服务器。
2. 三台计算层服务器:`sasapp01、sasapp02、sasapp03`，其他`sasapp01`为计算层主节点服务器，`sasapp02`和`sasapp03`为计算层Node节点服务器。
3. 三台中间层服务器:`sasmid01、sasmid02、sasmid03`，其中`sasmid01`为中间层主节点服务器，`sasmid02`和`sasmid03`为中间层Node节点服务器。

下面将分步介绍如何为上面部署环境完成一键启动配置配置。

1. 配置服务器间的SSH互信
2. 编写停启SAS Server的脚本
5. 配置开机启动SAS
6. 配置定时启停SAS


### 配置服务器间的SSH互信

* 使用`sas`用户登录每台SAS服务器并执行以下脚本，为`sas`用户创建公钥和私钥

```
cd /home/sas
ssh-keygen -t rsa
```

![](/img/in-post/2017-04-07/ssh-keygen.png)

* 配置配置服务器互信 

假设把sasmeta01作为管理服务器来管理其他Linux服务器，则需要使用`sas`用户登录sasmeta01服务器，把上面为`sas`用户创建的公钥id_rsa.pub通过下面命令复制到其被管理服务器的`/home/sas/.ssh`目录。

```
scp /home/sas/.ssh/id_rsa.pub sasmeta02:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasmeta03:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasapp01:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasapp02:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasapp03:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasmid01:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasmid02:/home/sas/.ssh/authorized_keys
scp /home/sas/.ssh/id_rsa.pub sasmid03:/home/sas/.ssh/authorized_keys
```

### 编写通用ssh远程执行shell脚本
* 启动SAS服务脚本`/home/sas/sas.startAllServers.sh`

``` shell
#!/bin/sh

echo ‘warning:this command must be executed on SAS Admin Server(sasmeta01) via user sas’

startSAS="nohup SASConfig/Lev1/sas.servers start &"

date 
ehco 'Starting sasmeta01'
$startSAS

date 
ehco 'Starting sasmeta02'
ssh sas@sasmeta02 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'Starting sasmeta03'
ssh sas@sasmeta03 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'Starting sasmeta03'
ssh sas@sasapp01 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'Starting sasmeta03'
ssh sas@sasapp02 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'Starting sasmeta03'
ssh sas@sasapp03 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'Starting sasmeta03'
ssh sas@sasmid01 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'Starting sasmeta03'
ssh sas@sasmid02 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date
ehco 'Starting sasmeta03'
ssh sas@sasmid03 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"

date 
ehco 'All sas servers started.'
```

然后用sas用户为脚本sas.startAllServers.sh添加执行权限

``` shell
cd /home/sas
chmod +x sas.startAllServers.sh
```

上述步骤完成后，`sas`用户可以在服务器`sasmeta01`上通过命令`/home/sas/sas.startAllServers.sh`完成所有服务器上的SAS Server的启动。

* 停止SAS服务脚本`/home/sas/sas.startAllServers.sh`

``` shell
#!/bin/sh
echo "warning:this command must be executed on SAS Admin Server(sasmeta01) via user sas"
stopSAS="nohup SASConfig/Lev1/sas.servers stop &"

date
ehco ’Stopping All SAS Servers...'
ssh sas@sasmid03 "${stopSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasmid02 "${stopSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasmid01 "${stopSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasapp03 "${stopSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasapp02 "${stopSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasapp01 "${stopSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasmeta03 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"
ssh sas@sasmeta02 "${startSAS} 0</dev/null 1>/dev/null 2>/dev/null"
$stopSAS

date
ehco 'All SAS Servers Stopped'
```

### 配置开机启动SAS

使用`root`用户登录SAS管理服务器(sasmeta01),修改文件`/etc/rc.d/rc.local`,添加下面记录配置开机启动SAS Server。

> sudo -u sas /home/sas/sas.startAllServers.sh

**注意：SAS服务器启动顺序，管理服务器(sasmeta01)必须最后一个启动**。

> 配置开机自动启动应该算是一个比较鸡肋的功能，最后是又系统运维人员重启系统后手工执行启动SAS服务的脚本。

### 配置定时停启SAS Servers

对于某些数据分析类项目，在数据准备窗口，为保证数据分析的可靠性，一般都会停止SAS计算层服务器，然后等待待分析的数据准备好后，再启动SAS计算层服务器执行数据分析工作。

> 比如Marketing Automation项目，Data Market一般会在每日0点到2点之间完成前一日业务数据统计计算，在这个时间段，应该停止所有SAS计算工作。

再Linux环境下，可以通过配置`/etc/crontab`来为每台需要定时停启的服务器配置停启任务。如上述需求，可以通过`root`用户在SAS计算层服务器`sasapp01、sasapp02、sasapp03`配置定时停启定时任务。配置内容如下

``` shell
50 23 * * * sas /SASConfig/Lev1/sas.servers stop
50 1 * * * sas /SASConfig/Lev1/sas.servers start  
```




