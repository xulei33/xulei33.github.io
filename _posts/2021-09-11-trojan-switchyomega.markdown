---
layout:     post
title:      "Trojan和SwitchyOmega手工安装配置说明"
subtitle:   "使用VPS部署Trojan和SwitchyOmega，观看YouTube"
date:       2021-09-11 12:00:00
author:     "Xulei"
header-img: "img/tag-bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Trojan
    - SwitchyOmega
---

## Trojan安装配置分为下面7个步骤，需要按照顺序执行

```
申请海外VPS，包含外部IP地址（AWS，Azure，阿里云国际等）
申请海外域名，配置DNS解析到VPS的外部IP地址上
安装nginx，配置静态网站，通过http可访问域名
申请安装https数字证书(通过acme.sh申请安装zerossl证书)
安装Trojan，配置系统启动服务，配置域名和密码，配置https证书
下载安装客户端，配置服务器信息，编写启停脚本
Edge浏览器安装SwitchyOmega插件，配置proxy，配置auto switch策略

```


## 安装VPS环境

```
yum -y install wget    #安装 wget
yum -y install curl    #安装 Curl 方法
yum -y install xz    #安装 XZ 压缩工具
yum update  #CentOS更新
```

## 安装NGINX
```
yum -y install  nginx wget unzip zip curl tar   #CentOS安装
systemctl enable nginx.service    #设置Nginx开机启动
```


## 配置Nginx
```
找到路径 /etc/nginx/nginx.conf 更新下面参数
user  root;
#tcp_nopush     on;
keepalive_timeout  120;
server_name  申请的域名; #申请的域名
```

### 设置伪装站点
```
rm -rf /usr/share/nginx/html/*   #删除目录原有文件
cd /usr/share/nginx/html/    #进入站点更目录
wget https://github.com/V2RaySSR/Trojan/raw/master/web.zip
unzip web.zip
systemctl restart nginx.service
```

到这里，你访问http://申请的域名 ，应该可以打开网站了。（不是https://）

## 下载Trojan服务器
点击访问Trojan服务端下载页面：[Releases · trojan-gfw/trojan · GitHub](https://github.com/trojan-gfw/trojan/releases) 查看最新版本地址
cd /usr/src  #进入该目录

```
wget https://github.com/trojan-gfw/trojan/releases/download/v1.16.0/trojan-1.16.0-linux-amd64.tar.xz    #下载官方Trojan服务器
tar xf trojan-1.*   #解压该文件
```

## 创建Trojan服务器配置文件
找到 /usr/src/trojan/ 目录，创建并打开server.conf文件，写入以下内容

```
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "00000000"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/usr/src/trojan-cert/fullchain.cer",
        "key": "/usr/src/trojan-cert/private.key",
        "key_password": "密码",
        "cipher_tls13":"TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
	"prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "no_delay": true,
        "keep_alive": true,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```

## 创建Trojan自启服务
Debian系统找到/lib/systemd/system/目录，并创建trojan.service文件
CentOS系统找到/usr/lib/systemd/system/目录，并创建trojan.service文件
打开trojan.service文件，并写入以下代码
```
[Unit]  
Description=trojan  
After=network.target  
   
[Service]  
Type=simple  
PIDFile=/usr/src/trojan/trojan/trojan.pid
ExecStart=/usr/src/trojan/trojan -c "/usr/src/trojan/server.conf"  
ExecReload=  
ExecStop=/usr/src/trojan/trojan  
PrivateTmp=true  
   
[Install]  
WantedBy=multi-user.target
```

## 开始申请SSL证书
HTTPS证书更新配置工具acme.sh
说明 · [acmesh-official/acme.sh Wiki · GitHub](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

```
curl  https://get.acme.sh | sh -s email=xulei33@msn.com
```

## 申请证书：
```
acme.sh --issue  -d 申请的域名   --nginx
```

得到证书
```
[Sun Sep 12 10:15:33 CST 2021] Your cert is in: /root/.acme.sh/申请的域名/申请的域名.cer
[Sun Sep 12 10:15:33 CST 2021] Your cert key is in: /root/.acme.sh/申请的域名/申请的域名.key
[Sun Sep 12 10:15:33 CST 2021] The intermediate CA cert is in: /root/.acme.sh/申请的域名/ca.cer
[Sun Sep 12 10:15:33 CST 2021] And the full chain certs is there: /root/.acme.sh/申请的域名/fullchain.cer
[root@v ~]# cd /root/.acme.sh/申请的域名/
```


## 移动证书文件
创建存放证书的文件夹trojan-cert 完整路径为 /usr/src/trojan-cert
```
[root@v 申请的域名]# mkdir /usr/src/trojan-cert
[root@v 申请的域名]# cp fullchain.cer /usr/src/trojan-cert/
[root@v 申请的域名]# cp 申请的域名.key /usr/src/trojan-cert/private.key
```

## 启动Trojan服务
设置启动Trojan服务
```
systemctl start trojan.service  #启动Trojan
systemctl enable trojan.service  #设置Trojan服务开机自启
```

## 验证SSL证书
访问 https://申请的域名 ，检查证书和Trojan是否正常运行（有小锁成功）。记得是访问 https:// 不是 http://

## 下载Trojan客户端软件
点击访问官方[Releases · trojan-gfw/trojan · GitHub](https://github.com/trojan-gfw/trojan/releases)，查看最新版本
下载客户端 trojan-1.16.0-win.zip

Win系统不想要Trojan的小黑窗，可以创建如下批处理运行Trojan.exe文件
命令如下

### start.bat 启动Trojan服务

```
@ECHO OFF
%1 start mshta vbscript:createobject("wscript.shell").run("""%~0"" ::",0)(window.close)&&exit
start /b trojan.exe
```

### stop.bat 停止Trojan服务

```
@ECHO OFF
taskkill /im trojan.exe /f
ping -n 2 127.1 >nul
```
把上述两个.bat文件，放在Trojan文件夹根目录下面，保证和Trojan.exe位于同一目录。

### 配置Trojan客户端
找到Trojan文件夹里面的config.json文件
修改下面两项参数，其余参数保持不变。

```
"remote_addr": "申请的域名",
    "password": [
        "密码"
    ],
```
双击start.bat启动客户端，监听本地1080端口数据。


## Windows10设置安装客户端开机启动
win+R打开允许，执行shell:startup打开启动文件目录，
复制start.bat的快捷方式到启动目录。

<img src="/img/in-post/2021-09-11/run.png" style="zoom:50%;" />

## 安装SwitchyOmega浏览器插件

访问Microsoft Edge Add-ons，查找SwitchyOmega，安装最新版本
Proxy SwitchyOmega - Microsoft Edge Addons


## 配置SwitchyOmega浏览器插件

<img src="/img/in-post/2021-09-11/proxy.png" style="zoom:50%;" />

<img src="/img/in-post/2021-09-11/auto.png" style="zoom:50%;" />


## 使用Trojan访问YouTube网站

<img src="/img/in-post/2021-09-11/youtube.png" style="zoom:50%;" />
