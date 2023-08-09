---
title: "Docker常用命令"
date: 2023-07-31T12:05:25+08:00
tags: ["Linux运维", "Docker"]
categories: []
draft: false
---

[Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice/)  
[Docker Guide](https://jiajially.gitbooks.io/dockerguide/content/)  
[清理Docker的container，image与volume · 零壹軒·笔记](https://note.qidong.name/2017/06/26/docker-clean/)  

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
docker rm -v $cid #同时删除 /var/lib/docker 目录下的数据卷
docker logs $cid #查看容器日志

docker system df #查看docker缓存大小
docker builder prune #一键清理 Build Cache
docker builder prune --filter 'until=240h' #保留最近10天的缓存
docker tag ca1b6b825289 devincpp/rk3568:v1.0 #重命名镜像
docker commit -a "author" -m "description" a404c6c174a2 devincpp/mysql:v1  #-p表示在提交时暂停容器

#数据卷是被设计用来持久化数据的，生命周期独立于容器，Docker不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收机制。
docker image prune #清理悬挂镜像
docker volume prune #清理无用数据卷
docker volume list #查看数据卷
docker volume create v2 #创建一个名为v2的数据卷，之后docker run时可以作为本地目录映射
docker volume rm v2 #删除一个名为v2的数据卷
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