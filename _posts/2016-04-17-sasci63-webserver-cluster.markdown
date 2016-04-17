---
layout:     post
title:      "SAS WebServer Cluster Configuration"
subtitle:   "SASCI6.3 high availability deployment"
date:       2016-04-16 13:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - SAS
    - SASCI63
---

# SAS WebServer Cluster

对系统有高可用要求的情况下，SAS的默认部署计划是不能满足要求，所以我们需要在通过SAS Deployment Wizard使用默认部署计划完成系统部署后，还需要对系统进行手工配置。

>
> SAS6.3默认部署计划中，SAS WebServer只部署在中间层主节点上，中间层的Node节点上只有WebAPPServer,这样就导致了如果中间层主节点服务器停机的话，即使Node节点工作正常，用户的请求也不能通过WebServer发送到Node节点的WebAPPServer，系统部署架构存在单点问题。

在中间层Node服务器部署SASWebServer有下面3个步骤：

1. Install SAS WebServer
2. Configure SAS WebServer
3. Configure Load Balancer

## Install SAS WebServer
使用XManager通过SAS部署用户·sas·远程登录中间层Node服务器，`cd /share/SASSoftDepot/`,通过命令`./setup.sh`执行SAS Deployment Wizard。

![](/img/in-post/2017-04-07/2016-04-17 (1).png)

Choose `Install SAS Software`,then click `Next` to continue
![](/img/in-post/2017-04-07/2016-04-17 (2).png)

Choose`Install Additional Software`,then click `Next` to continue
![](/img/in-post/2017-04-07/2016-04-17 (3).png)

UnCheck all other SAS softwares, only check `SAS WebServer`,then click `Next` to install SAS WebServer on current server. 

安装完成后，不要使用SAS Deployment Wizard对服务器进行自动·configure·，请参照下面部署进行手工配置。

## Configure SAS WebServer
###复制SASWebServer
使用SAS系统部署用户登录到中间层主节点服务器，
·
cd SASConfig/Lev1/Web/
tar czvf WebServer.tar.gz WebServer
·
然后使用SAS系统部署用户·sas·复制WebServer.tar.gz到Node节点服务器的·SASConfig/Lev1/Web/·目录。
Redhat下可以使用scp命令 ·scp WebServer.tar.gz sas@NodeServerIP:SASConfig/Lev1/Web/WebServer.tar.gz·
文件复制完成后，使用命令` tar xzvf WebServer.tar.gz `解压文件。

###配置conf和sas.conf
使用SAS系统部署用户`sas`登录中间层Node节点服务器，`cd SASConfig/Lev1/Web/WebServer/conf/`，然后把`httpd.conf`里`ServerName:7980`由中间层主节点修改为Node节点的服务器名字。
`NodeServerName:7980`

> 查看服务器名称shell命令 `hostname`

以上步骤配置好后，通过`./sas.servers start`命令启动中间层Node服务器时，就可以看到WebServer已经在服务器启动列表里了。
然后通过浏览器，通过地址`http://NodeServerIP:7980/SASCIStudio`登录系统时，可以发现浏览器会被聪定向到`http://中间层主节点IP:7980/SASCIStudio`，
所以WebServer的Cluster配置还没有完成，需要在WebServer前端在配置上负载均衡器（F5或者DNS），配置步骤见下面。

## Configure Load Balancer

SAS服务器配置完成后，默认对外提供服务的的地址是自动安装的WebServer服务器的IP地址（即中间层主节点服务器的IP地址），这些配置信息分布在SAS元数据和服务器配置文件中，这些都需要修改。

### F5

> 负载均衡设备需要配置为支持粘性会话（stick Session），SAS中间层服务器之间还没有支持会话共享，用户从一个服务器切换到另外一个服务器时，需要重新登录。

##### 配置文件修改

###### server.xml
使用SAS系统部署用户`sas`登录所有`中间层服务器`，`cd /SASConfig/Lev1/Web/WebAppServer`,对目录下所有SASServer（SASServer1_1,SASServer2_1,SASServer6_1...)下的`/conf/server.xml`的下面内容进行修改。

```
<connection ... proxyName="F5IP或DNS域名" port="80(如果F5有做端口映射)或7980" .../> 
```

###### 修改JVM Opt
使用SAS系统部署用户`sas`登录所有中间层服务器，`vi SASConfig/Lev1/Web/WebAppServer/SASServer1_1/bin/setenv.sh`，在JVM option后面添加下面内容

```
-Dsas.scs.cas.scheme=http -Dsas.scs.cas.host=F5IP或DNS域名 -Dsas.scs.cas.port=80或7980 -Dsas.scs.svc.scheme=http -Dsas.scs.svc.host=F5IP或DNS域名 -Dsas.scs.svc.port=80或7980
```

###### 修改SASEnvironmentManager配置文件
使用SAS系统部署用户`sas`登录中间层主节点服务器，修改`security-web-context.xml`和`web.xml`内的地址,由`中间层主节点服务器IP`修改为`F5地址或者DNS域名`。文件路径为

1. SASConfig/Lev1/Web/SASEnvironmentManager/server-5.0.0-EE/hq-engine/hq-server/webapps/ROOT/WEB-INF/web.xml
2. SASConfig/Lev1/Web/SASEnvironmentManager/server-5.0.0-EE/hq-engine/hq-server/webapps/ROOT/WEB-INF/spring/security-web-context.xml

##### 元数据修改

###### 修改Application的connection
使用SMC通过用户sasadm@saspw登录元数据服务器，在Application Management下的Configuration Manage下有很多SAS Application，如Customer Intelligence Core 6.3等，打开这个应用程序的属性配置界面，切换到Internal Connection选项卡，然后修改里面的Host Name，把默认的中间层主节点的地址修改为F5的地址或者DNS域名，如果F5有做端口映射，也要在这里修改Port Number，从默认的7980修改为F5映射的地址，如80。
![](/img/in-post/2017-04-07/2016-04-17 (4).png)

###### 修改SAS Content Server
使用SMC通过用户sasadm@saspw登录元数据服务器，在Server Manager下面找到SAS Content Server，打开属性页面的option页面，修改里面的Host name、Port detail和Proxy的URL。
把默认的中间层主节点的地址修改为F5的地址或者DNS域名，如果F5有做端口映射，把默认的7980端口修改F5映射的端口。
![](/img/in-post/2017-04-07/2016-04-17 (5).png)

###### 修改WebDAV连接
使用SMC通过用户sasadm@saspw登录元数据服务器，在Environment Management下的Foundation Service Manager下有很多SAS基础服务，需要参数下图对所有的SAS基础服务的Information Service的WebDav连接进行修改，把默认的中间层服务器的地址和端口修改为F5(或DNS域名)和F5映射的新端口(80端口)。

![](/img/in-post/2017-04-07/2016-04-17 (6).png)
![](/img/in-post/2017-04-07/2016-04-17 (7).png)

###### 修改MaLauncher
使用SAS系统部署用户`sas`登录计算层服务器，编辑文件`vi SASHome/SASMarketingAutomationLauncher/6.3/spring-config/application.properties`，

> 修改`${webinfpltfm.client.registry.url}`指向的URL地址，由中间层主节点服务器地址修改为F5地址或者DNS域名，如果F5设置了端口映射，也同时修改URL中的端口号。

## Restart All SAS Servers

以上步骤配置好后，需要通过用户`sas`按照从中间层、计算层、元数据层的顺序停止所有服务器，

> 停止SAS服务器命令`./sas.servers stop`。

所有服务器都停止完成后，在通过用户`sas`，安装元数据层、计算层、中间层顺序，逐台启动sas服务器

> 启动命令`./sas.servers start`。
