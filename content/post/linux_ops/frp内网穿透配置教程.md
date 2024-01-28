---
title: "frp内网穿透配置教程"
date: 2023-08-04T16:05:25+08:00
tags: ["Linux运维", "frp"]
categories: []
draft: false
toc: true
---

# 重要资料
[fatedier/frp - github](https://github.com/fatedier/frp)  
[官方文档 | frp](https://gofrp.org/docs/)  
[司波图/自建内网穿透服务器 - 码云](https://gitee.com/spoto/natserver)  

```
$ tree /opt/frp
/opt/frp
├── frpc
├── frpc_full.ini
├── frpc.ini
├── frps
├── frps_full.ini
├── frps.ini
└── LICENSE
```

# 1.SSH访问内网机器
[通过 SSH 访问内网机器 | frp](https://gofrp.org/docs/examples/ssh/)  
## 1.1.Server配置
在具有公网 IP 的机器上部署 frps，修改 frps.ini 文件，这里使用了最简化的配置，设置了 frp 服务器用户接收客户端连接的端口。  
```ini
#/opt/frp/frps.ini
[common]
bind_port = 7000 # 注意：需要在云服务设置开放7000端口
# 身份验证(可选)
token = i*RY2KI9^A7H
# web界面
dashboard_port = 7500
# dashboard 用户名密码，可选，默认为空
dashboard_user = admin
dashboard_pwd = admin
```
运行方式：`/opt/frp/frps -c /opt/frp/frps.ini`

## 1.2.Client配置
在需要被访问的内网机器上（SSH 服务通常监听在 22 端口）部署 frpc，修改 frpc.ini 文件，假设 frps 所在服务器的公网 IP 为 x.x.x.x。  
local_ip 和 local_port 配置为本地需要暴露到公网的服务地址和端口。remote_port 表示在 frp 服务端监听的端口，访问此端口的流量将会被转发到本地服务对应的端口。
```ini
#/opt/frp/frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000
# 身份验证(可选)
token = i*RY2KI9^A7H

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000 # 注意：需要在云服务器上开放6000端口
```

运行方式：`/opt/frp/frpc -c /opt/frp/frpc.ini`  
公网机器ssh访问内网机器的方式：`ssh -p 6000 firefly@localhost`  
公网机器scp访问内网机器的方式：`scp -P 6000 ./* firefly@localhost:/opt`  

注意：不要使用后台方式运行，`nohup /opt/frp/frpc -c /opt/frp/frpc.ini 2>&1 &`
，否则下次连接可能会遇到以下报错：[frp错误，frp报错，[ssh] start error: proxy name [ssh] is already in use_狗狗25的博客-CSDN博客](https://blog.csdn.net/wzying25/article/details/105482746)

## 1.3.设置frps开机自启动
[使用 systemd | frp](https://gofrp.org/docs/setup/systemd/)  

```ini
$ sudo vim /etc/systemd/system/frps.service
$ systemctl cat frps
# /etc/systemd/system/frps.service
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /opt/frp/frps -c /opt/frp/frps.ini

[Install]
WantedBy = multi-user.target
```
```shell
# 配置 frps 开机自启
systemctl enable frps
# 启动frp
systemctl start frps
# 停止frp
systemctl stop frps
# 重启frp
systemctl restart frps
# 查看frp状态
systemctl status frps
```


## 1.4.设置frpc开机自启动
```shell
$ sudo systemctl cat frpc
# /etc/systemd/system/frpc.service
[Unit]
# 服务名称，可自定义
Description = frp client
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStartPre = /bin/sleep 10
ExecStart = /opt/frp/frpc -c /opt/frp/frpc.ini

[Install]
WantedBy = multi-user.target
```

```shell
$ systemctl get-default
graphical.target
$ cd /etc/systemd/system
$ sudo vim frpc.service
$ sudo systemctl enable frpc
Created symlink /etc/systemd/system/multi-user.target.wants/frpc.service → /etc/systemd/system/frpc.service.
$ sudo systemctl status frpc
● frpc.service - frp client
   Loaded: loaded (/etc/systemd/system/frpc.service; enabled; vendor preset: enabled)
   Active: inactive (dead)
$ sudo systemctl restart frpc
$ sudo systemctl status frpc
● frpc.service - frp client
   Loaded: loaded (/etc/systemd/system/frpc.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2023-08-06 05:13:52 UTC; 8s ago
 Main PID: 1361 (frpc)
   CGroup: /system.slice/frpc.service
           └─1361 /opt/frp/frpc -c /opt/frp/frpc.ini

Aug 06 05:13:52 firefly systemd[1]: Started frp client.
Aug 06 05:13:52 firefly frpc[1361]: 2023/08/06 05:13:52 [I] [root.go:220] start frpc service for config file [/opt/frp/frpc.ini]
Aug 06 05:13:53 firefly frpc[1361]: 2023/08/06 05:13:53 [I] [service.go:301] [f8d3384fbb8c1471] login to server success, get run id [f8d3384fbb8c1471]
Aug 06 05:13:53 firefly frpc[1361]: 2023/08/06 05:13:53 [I] [proxy_manager.go:150] [f8d3384fbb8c1471] proxy added: [ssh]
Aug 06 05:13:53 firefly frpc[1361]: 2023/08/06 05:13:53 [I] [control.go:172] [f8d3384fbb8c1471] [ssh] start proxy success
```

# 2.Systemd
[Systemd 入门教程：命令篇 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)  

## 2.1 service命令
`service`其实是去`/etc/init.d`目录下执行相关程序
```bash
# service命令启动redis脚本
service redis start
# 直接启动redis脚本
/etc/init.d/redis start
# 开机自启动
update-rc.d redis defaults
# 查看服务状态
service redis status
```
## 2.2 systemctl命令
systemd是Linux系统最新的初始化系统(init)，作用是提高系统的启动速度，尽可能启动较少的进程，尽可能更多进程并发启动。systemd对应的进程管理命令是systemctl

systemctl命令兼容了service，即systemctl也会去/etc/init.d目录下，查看执行相关程序。  
systemctl命令管理systemd的资源Unit放在目录`/usr/lib/systemd/system`(Centos)或`/etc/systemd/system`(Ubuntu)，主要有四种类型文件.mount,.service,.target,.wants
```bash
systemctl redis start
systemctl redis stop
systemctl enable redis
systemctl status redis
```
