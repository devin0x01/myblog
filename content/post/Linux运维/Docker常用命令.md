---
title: "Docker常用命令"
date: 2023-07-31T12:05:25+08:00
tags: ["Linux运维", "Docker"]
categories: []
draft: false
---

# docker
```shell
docker info #docker配置信息
docker inspect $cid #查看容器的配置信息

docker images
docker ps -a
docker run -it $image_id --rm  #rm表示退出容器后就删除该容器
docker exec -it $cid /bin/bash

docker start $cid
docker stop  $cid
docker restart $cid
docker rm -f $cid
docker logs $cid #查看容器日志

docker system df #查看docker缓存大小
docker builder prune #一键清理 Build Cache
docker builder prune --filter 'until=240h' #保留最近10天的缓存
docker tag ca1b6b825289 devincpp/rk3568:v1.0 #重命名镜像
docker commit -a "author" -m "description" a404c6c174a2 devincpp/mysql:v1  #-p表示在提交时暂停容器
```

# docker compose
[docker compose build | Docker Documentation](https://docs.docker.com/engine/reference/commandline/compose_build/)  
[Docker Compose | 菜鸟教程](https://www.runoob.com/docker/docker-compose.html)

```shell
#docker compose只能识别yaml后缀，yml文件也不能识别，需要加-f
docker compose -f <your_file.yml> ...
#前台展示启动
docker compose up
#后台启动
docker compose up -d
#停止并删除容器服务
docker compose down
#列出所有运行容器
docker compose ps
#查看服务日志
docker compose logs
#构建或者重新构建服务
docker compose build
#启动服务
docker compose start
#停止已运行的服务
docker compose stop
#重启服务
docker compose restart
```