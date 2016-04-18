---
layout:     post
title:      "SAS作业预定和执行调度部署"
subtitle:   "SAS Platform Process Manager & Platform LSF "
date:       2016-04-17 21:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - SAS
    - Linux
---

# 说明

SAS Platform Suite里的Platform Process Manager和Platform LSF是SAS推荐使用的作业预定和执行调度管理软件，它们既可以和SAS Grid Manager一起组成SAS高性能网格计算，也可以在客户不需要SAS高性能网格计算时，作为作业预定和执行调度管理软件单独部署。

>
> * Platform Suite for SAS包含三个组件：Process Manager、LSF和Grid Management Service（简称GMS）。GMS和SAS HPA由于License费用较高，一般企业用户不会用到，更多情况是Platform Manager和Platform LSF一起作为作业预定和执行调度软件单独部署使用。
> * SAS除了提供Platform Process Manager做为作业预定管理器外，还提供OS Schedule、in-memory schedule和distribute in-memory schedule，除了standalone部署架构推荐使用OS schedule外，建议其他部署架构使用更加可靠的Platform Process Manager作为作业预定管理器。
> * Platform LSF有两种安装方式：方式一：独立安装 ；方式二：在Process Manager安装后自动安装。两种安装方式在Platform LSF的安装过程中没有差别。但推荐大家采用更方便的第二种安装方式。
>

## 安装说明
Process Manager作为作业预定管理器，默认只需要安装在计算层主节点上，Platform LSF则需要在所有计算层服务器上都安装。现假设有4台Redhat6.4计算服务器需要配置：
`sasapp01、sasapp02、sasapp03、sasapp04`，其中`sasapp01`为计算层主节点服务器，`sasapp02`、`sasapp03`和`sasapp04`为计算层Node节点服务器。

> Platform Process Manager支持standby模式的高可用，可以选择在部署第一台服务器时设置`EGO_DAEMON_CONTROL=true`，完成部署后再手工调整`js.conf`文件，然后再第二台计算层服务器上部署Process Manager。这样可以在第一台Process Manager不可用时，作业预定管理服务可以由第二台服务器上的Process Manager接管。

下面将分步介绍如何为上面部署环境配置Platform Process Manager和Platform LSF。

1. 安装前准备
2. Process Manager和LSF安装
3. Process Manager配置
4. Platform LSF配置
5. SAS元数据配置


### 安装前准备



