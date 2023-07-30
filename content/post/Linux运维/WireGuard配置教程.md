---
title: "WireGuard配置教程.md"
date: 2023-07-30T12:10:05+08:00
tags: ["Linux运维", "Docker"]
categories: []
draft: false
---

[Run WireGuard VPN Server in Docker Container with Docker Compose - TechViewLeo](https://techviewleo.com/run-wireguard-server-in-docker-container/?expand_article=1) -- 主要参考这个  
[基于Wireguard技术的虚拟个人网络搭建: 基于wireguard的内网穿透技术~](https://gitee.com/spoto/wireguard)  
[搭建WireGuard-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2233314)

- 公网机器: IP=100.101.102.103, Name=TencentVM1
- 私网机器: IP=192.168.123.189, Name=LocalMint1

# 1.公网配置
## 1.1.安装Docke和docker-compose
[Docker Compose | 菜鸟教程](https://www.runoob.com/docker/docker-compose.html)

```shell
# 安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.19.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod 755 /usr/local/bin/docker-compose
docker-compose version
```

## 1.2.安装linuxserver/wireguard镜像
```shell
sudo mkdir /opt/wireguard-server
vim docker-compose.yaml # yaml里需要配置容器的名字，server的地址
docker compose up -d

# 检查WireGuard服务器的状态
docker exec -it wireguard wg
docker exec -it wireguard /bin/bash

# 其他docker compose命令
docker compose start wireguard
docker compose restart wireguard
docker compose ps
```

配置文件的目录结构
```shell
ubuntu@VM-4-3-ubuntu:/opt/wireguard-server $ tree
.
├── config
│   ├── coredns
│   │   └── Corefile
│   ├── peer1
│   │   ├── peer1.conf
│   │   ├── peer1.png
│   │   ├── presharedkey-peer1
│   │   ├── privatekey-peer1
│   │   └── publickey-peer1
│   ├── server
│   │   ├── privatekey-server
│   │   └── publickey-server
│   ├── templates
│   │   ├── peer.conf
│   │   └── server.conf
│   └── wg0.conf
└── docker-compose.yaml

5 directories, 12 files
```

`/opt/wireguard-server/docker-compose.yaml`内容如下，需要修改`SERVERURL`字段  
`PEERS=8`时会生成8个peer的配置文件
>注意: WireGuard 使用的是UDP协议，下面的配置文件使用的端口是51820
```yaml
version: '3.7'
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Nairobi #set correct timezone
      - SERVERPORT=51820 #optional
      - PEERS=1 #optional
      - PEERDNS=auto #optional
      - ALLOWEDIPS=0.0.0.0/0 #Peer addresses allowed
      - INTERNAL_SUBNET=10.13.13.0/24 #Subnet used in VPN tunnel
      - SERVERURL=100.101.102.103 #Wireguard VPN server address
    volumes:
      - /opt/wireguard-server/config:/config
      - /usr/src:/usr/src # location of kernel headers
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: always
```

## 1.3.防火墙设置
```shell
### on Redhat Based ###
sudo firewall-cmd --permanent --add-port=51820/udp
sudo firewall-cmd --reload

### On Debian Based ###
sudo apt install ufw
sudo ufw allow 51820/udp
```

![腾讯云服务器防火墙设置](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_48ff7bf1ae7a6ae1fa4979f8fecfccec.png)

# 2.私网配置
## 2.1.安装wireguard
[wireguard已合入内核5.6版本 - 博客园](https://www.cnblogs.com/yangtao416/p/16372660.html)  
```shell
sudo apt install -y wireguard openresolv
```

## 2.2.拷贝配置文件到LocalMint1
```shell
# 拷贝配置文件
scp /opt/wireguard-server/config/peer1/peer1.conf username@serverIP:~/peer1.conf
sudo mv ~/peer1.conf /etc/wireguard/wg0.conf

# 设置服务开机自启动
sudo systemctl enable wg-quick@wg0
sudo reboot

# 检查状态
systemctl status wg-quick@wg0
ip ad
ifconfig
ping 10.13.13.1

# 这时候可以在server段检查下状态，会发现peer已经连接上去了
docker exec -it wireguard wg
```

# 3.添加更多的节点
```shell
# Edit on the server side
$ sudo vim /opt/wireguard-server/config/wg0.conf
# peer2
PublicKey = 7ANB0SuBUsnetjqHrL99YIhpbqetJ9yYy0CRsNiuzls=
PresharedKey = gbMDUgQM7levlYLcwhyf1E1dHF/PG489UGeeSHr7tro=
AllowedIPs = 10.13.13.3/32
```