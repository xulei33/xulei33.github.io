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

多层部署架构下的SAS软件通常分布在多台服务器上，如元数据服务器至少会部署在3台服务器上，计算层服务器根据分析计算的数据量规模不同可能会部署几台到几十台服务器上，WebServer和WebAPPServer会根据系统使用人员数量至少部署在两台服务器上。另外SAS软件对系统启停还有一些特别要求，这给系统运维人员带了非常大的麻烦。

>
> 启动SAS服务器，需要使用用户`sas`执行`SASConfig/Lev1/sas.servers start`命令来启动SAS Server。需要按照`元数据主节点、元数据Node节点、计算层主节点、计算层Node节点、WebServer、中间层主节点、中间层Node节点服务器`的顺序启动SAS Server，其中计算层Node节点服务器可以并发启停，其他服务器需要用户等前一台服务器完成启动后，才能再启动下一台服务器。
>
> 停止SAS服务的步骤同启动相反,需要使用系统用户`sas`执行`SASConfig/Lev1/sas.servers stop`命令来停止SAS Server。 中间层Node节点服务器和计算层Node节点服务器可以并发停止，其他服务器要等前一台停止完成后才能停止下一台。

在Linux环境下，通过配置服务器间的ssh互信，可以达到通过shell在一台服务器上执行其他服务器上shell的功能，这样我们可以通过编写一个shell脚本通过ssh命令来完成SAS多台服务器的启停工作，然后在通过配置开机启动和定时重启等，可以大大减少系统运维人员的工作量。下面将分六块分别介绍如何完成这些配置。

1. 配置服务器间的SSH互信，通过配置ssh互信，可以达到在调用ssh命令是不要输入远程服务启动用户名和密码。
2. 编写通用ssh远程执行shell脚本
3. 编写SAS启动脚本
4. 编写SAS停止脚本
5. 配置开机启动SAS
6. 配置定时启停SAS

## 配置服务器间的SSH互信

