---
title: "自建RSS服务"
date: 2023-08-08T16:05:25+08:00
tags: ["RSS"]
categories: []
draft: false
---

[我的 RSS 最佳实践](https://slarker.me/rss-best-practices/)

# 1.部署Fresh RSS
[笔记｜Docker 快速搭建 FreshRSS | Jack‘s Space](https://veryjack.com/technique/docker-install-freshrss/)  
[Docker 时区调整方案-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1626811)  

Android可以使用Feedme客户端，Windows可以通过`ip:port`方式访问  
```shell
docker run -d --restart unless-stopped --log-opt max-size=10m \
  -p 8282:80 \
  -e TZ=Asia/Shanghai \
  -e 'CRON_MIN=1,15,31' \
  -v /opt/freshrss/data:/var/www/FreshRSS/data \
  -v /opt/freshrss/extensions:/var/www/FreshRSS/extensions \
  --name freshrss \
  freshrss/freshrss
```

# 2.部署Tiny Tiny RSS
[docker-compose 部署 RSS 服务订阅、安装tiny-tiny-rss、RSSHub - 贝尔塔猫 - 博客园](https://www.cnblogs.com/CyLee/p/16159637.html)  
```shell
# 下载 docker-compose.yml 配置文件
mkdir -p /opt/ttrss && cd /opt/ttrss
curl -fLo docker-compose.yml https://raw.githubusercontent.com/HenryQW/Awesome-TTRSS/main/docker-compose.yml
```

```shell
# 删除 Docker 容器
docker-compose down
# 删除已停止的 Docker 容器
docker-compose rm
# 开启 Docker 服务
docker-compose up -d
```

默认账户：admin/password  
访问地址：配置文件中的`SELF_URL_PATH`字段  

## 2.1.docker-compose.yml 配置文件
```yaml
version: "3"
services:
  service.rss:
    image: wangqiru/ttrss:latest
    container_name: ttrss
    ports:
      - 12345:80
    environment:
      - SELF_URL_PATH=http://100.101.102.103:12345 # please change to your own domain
      - DB_PASS=ttrss # use the same password defined in `database.postgres`
      - PUID=1000
      - PGID=1000
    volumes:
      - feed-icons:/var/www/feed-icons/
    networks:
      - public_access
      - service_only
      - database_only
    stdin_open: true
    tty: true
    restart: always

  service.mercury: # set Mercury Parser API endpoint to `service.mercury:3000` on TTRSS plugin setting page
    image: wangqiru/mercury-parser-api:latest
    container_name: mercury
    networks:
      - public_access
      - service_only
    restart: always

  service.opencc: # set OpenCC API endpoint to `service.opencc:3000` on TTRSS plugin setting page
    image: wangqiru/opencc-api-server:latest
    container_name: opencc
    environment:
      - NODE_ENV=production
    networks:
      - service_only
    restart: always

  database.postgres:
    image: postgres:13-alpine
    container_name: postgres
    environment:
      - POSTGRES_PASSWORD=ttrss # feel free to change the password
    volumes:
      - /opt/ttrss/postgres/data/:/var/lib/postgresql/data # persist postgres data to ~/postgres/data/ on the host
    networks:
      - database_only
    restart: always

  # utility.watchtower:
  #   container_name: watchtower
  #   image: containrrr/watchtower:latest
  #   volumes:
  #     - /var/run/docker.sock:/var/run/docker.sock
  #   environment:
  #     - WATCHTOWER_CLEANUP=true
  #     - WATCHTOWER_POLL_INTERVAL=86400
  #   restart: always

volumes:
  feed-icons:

networks:
  public_access: # Provide the access for ttrss UI
  service_only: # Provide the communication network between services only
    internal: true
  database_only: # Provide the communication between ttrss and database only
    internal: true
```
