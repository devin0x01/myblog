---
title: "WireGuard配置教程"
date: 2023-07-30T12:10:05+08:00
tags: ["Linux运维", "WireGuard", "Docker"]
categories: []
draft: false
toc: true
---

[Run WireGuard VPN Server in Docker Container with Docker Compose - TechViewLeo](https://techviewleo.com/run-wireguard-server-in-docker-container/?expand_article=1) -- 主要参考这个  
[基于Wireguard技术的虚拟个人网络搭建: 基于wireguard的内网穿透技术~](https://gitee.com/spoto/wireguard)  
[搭建WireGuard-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2233314)

- 公网机器: IP=100.101.102.103, Name=TencentVM1
- 私网机器: IP=192.168.123.189, Name=LocalMint1

# 1.服务端配置
## 1.1.安装wg-easy镜像
[wg-easy/wg-easy: The easiest way to run WireGuard VPN + Web-based Admin UI.](https://github.com/wg-easy/wg-easy)  
[基于Wireguard技术的虚拟个人网络搭建: 基于wireguard的内网穿透技术~](https://gitee.com/spoto/wireguard#docker%E5%AE%89%E8%A3%85wireguard)  
[Wireguard 全互联模式（full mesh）配置指南 – 云原生实验室 - Kubernetes|Docker|Istio|Envoy|Hugo|Golang|云原生](https://icloudnative.io/posts/wireguard-full-mesh/)

这个新建的容器内会多一个`wg0`网卡，IP地址是`10.0.8.1`，而且网卡只在容器内可见。  
server的配置文件会放在`/opt/wg-easy`目录下。TCP的51821端口用于访问WebUI。  
```shell
docker run -d \
  --name=wg-easy \
  -e WG_HOST=123.123.123.123 (这里输入服务器的公网IP) \
  -e PASSWORD=passwd123 (这里输入你的密码) \
  -e WG_DEFAULT_ADDRESS=10.0.8.x （默认IP地址）\
  -e WG_DEFAULT_DNS=114.114.114.114 （默认DNS）\
  -e WG_ALLOWED_IPS=10.0.8.0/24 （允许连接的IP段）\
  -e WG_PERSISTENT_KEEPALIVE=25 （重连间隔）\
  -v /opt/wg-easy:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  weejewel/wg-easy
```

## 1.2.安装linuxserver/wireguard镜像(建议使用wg-easy)
```shell
# 安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.19.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod 755 /usr/local/bin/docker-compose
docker-compose version
```

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

### 1.2.1.配置文件的目录结构
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

### 1.2.2.服务端配置文件
```shell
#/opt/wireguard-server/config/wg0.conf

[Interface]
Address = 10.13.13.1
ListenPort = 51820
PrivateKey = UIx5/v...
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE

[Peer]
# peer1
PublicKey = x4iD6...
PresharedKey = OCjHe...
AllowedIPs = 10.13.13.2/32

[Peer]
# peer2
PublicKey = 91rwZ...
PresharedKey = /lOtI...
AllowedIPs = 10.13.13.3/32
```
**Interface**  
这个配置项为本地接口的配置，其中：  
Address 为 VPN 连接的本地 IP 地址  
ListenPort 作为服务端需要声明一个监听的端口，WireGuard 使用 UDP 协议，这个端口可以任意填写。需要保证防火墙已开放 UDP 的这个端口  
PrivateKey 为上一步生成的私钥

**Peer**  
这个为对端的配置，如果有多个，则需添加多个 Peer 配置，服务端的 Peer 配置项即定义了各个可连接的客户端。其中：  
PublicKey 为对端的公钥  
AllowedIPs 在配置路由会讲到  

### 1.2.3.客户端配置文件
```shell
#/opt/wireguard-server/config/peer1/peer1.conf

[Interface]
Address = 10.13.13.2
PrivateKey = kGaLz...
ListenPort = 51820
DNS = 10.13.13.1

[Peer]
PublicKey = T2i88...
PresharedKey = OCjHe...
Endpoint = 100.101.102.103:51820
AllowedIPs = 0.0.0.0/0
```
客户端与服务端不同的地方在于：  
Interface 配置中没有了 ListenPort  
Peer 即为服务端，与服务端不同的地方在于多了一个 Endpoint  

### 1.2.4.Docker Compose配置文件
```yaml
#/opt/wireguard-server/docker-compose.yaml
#需要修改`SERVERURL`字段  
#`PEERS=8`时会生成8个peer的配置文件

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
![腾讯云服务器防火墙设置](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_48ff7bf1ae7a6ae1fa4979f8fecfccec.png)

下面这个好像非必须？
```shell
### on Redhat Based ###
sudo firewall-cmd --permanent --add-port=51820/udp
sudo firewall-cmd --reload

### On Debian Based ###
sudo apt install ufw
sudo ufw allow 51820/udp
```


# 2.客户端配置
## 2.1.安装wireguard
### 2.1.1.内核版本大于5.6
```shell
sudo apt install -y wireguard openresolv
```

### 2.1.2.内核版本4.19
[WireGuard 白皮书带读3 - 知乎](https://zhuanlan.zhihu.com/p/478369772)  
```shell
sudo apt-get install -y wireguard-dkms wireguard-tools linux-headers-$(uname -r) 
sudo ip link add dev wg0 type wireguard
sudo ip address add dev wg0 192.168.1.1/32
sudo wg set wg0 listen-port 6789 private-key /etc/wireguard/privatekey
sudo ip link set wg0 up
sudo wg-quick up wg0
```
实际测试时报错如下，未解决:
```shell
$ sudo ip link add dev wg0 type wireguard
RTNETLINK answers: Operation not supported
```

## 2.2.拷贝配置文件并启动
### 2.2.1.VM类型客户端
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

`wg-quick up wg0`会自动去找配置文件`/etc/wireguard/wg0.conf`
```shell
$ systemctl cat wg-quick@wg0
# /lib/systemd/system/wg-quick@.service
[Unit]
Description=WireGuard via wg-quick(8) for %I
After=network-online.target nss-lookup.target
Wants=network-online.target nss-lookup.target
PartOf=wg-quick.target
Documentation=man:wg-quick(8)
Documentation=man:wg(8)
Documentation=https://www.wireguard.com/
Documentation=https://www.wireguard.com/quickstart/
Documentation=https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8
Documentation=https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/wg-quick up %i
ExecStop=/usr/bin/wg-quick down %i
Environment=WG_ENDPOINT_RESOLUTION_RETRIES=infinity

[Install]
WantedBy=multi-user.target
```

### 2.2.2.容器类型客户端
无法设置一些服务开机自启，比如无法执行`systemctl enable wg-quikc@wg0`
```shell
## 配置VPN，从wg-easy的WebUI下载配置文件
vim ~/wg0.conf
sudo wg-quick up ~/wg0.conf
#如果提示wg0存在时(wg-quick: `wg0' already exists)执行下面一行
#sudo ip link delete dev wg0
```

网卡相关的一些命令
```shell
sudo ip link add wg0 type wireguard
sudo ip link delete dev wg0
sudo ip link set up dev wg0
sudo ifconfig wg0 up
ifconfig
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
