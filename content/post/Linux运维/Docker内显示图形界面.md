---
title: "Docker内显示图形界面"
date: 2023-07-28T16:05:25+08:00
tags: ["Linux运维", "Docker", "Qt"]
categories: []
draft: false
---
# 关于X11
[x(7) - Linux man page](https://linux.die.net/man/7/x)  
[Cygwin系列（十二）：了解X - 知乎](https://zhuanlan.zhihu.com/p/134325713)  
>X11采用了C/S的架构，在其设计下，整个图形视窗系统主要分为3个部分：  
1.X Server（X服务器）。X Server一方面负责和设备驱动交互，监听显示器和键盘鼠标，另一方面响应X Client需求传递键盘、鼠标事件、（通过设备驱动）绘制图形文字等。反直觉之一，**X Server运行在本地**。  
2.X Client（X客户端）。X Client也叫X应用程序，负责实现程序逻辑，在收到设备事件后计算出绘图数据，由于本身没有绘制能力，只能向X Server发送绘制请求和绘图数据，告诉X Server在哪里绘制一个什么样的图形。X Client可以和X Server在同一个主机上，也可以通过TCP/IP网络连接。  
3.Window Manager（窗口管理器，简称WM），或者叫合成器（Compositor）。多个X Client向X Server发送绘制请求时，各X Client程序并不知道彼此的存在，绘制图形出现重叠、颜色干扰等问题是大概率事件，这就需要一个管理者统一协调，即Window Manager，它掌管各X Client的Window（窗口）视觉外观，如形状、排列、移动、重叠渲染等。反直觉之二，Window Manager并非X Server的一部分，而是一个特殊的X Client程序。  
3个部分， X Server是整个X Window System的中心，协调X客户端和窗口管理器的通信。

![X](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_c6d6319755f698570c734a5b2a6aad56.png)


# 常用命令
[xorg - What does $DISPLAY environment variable mean - Ask Ubuntu](https://askubuntu.com/questions/1284285/what-does-display-environment-variable-mean)  
[x11 forwarding - How to fix "MobaXterm X11 proxy: Unsupported authorisation protocol" - Super User](https://superuser.com/questions/1111900/how-to-fix-mobaxterm-x11-proxy-unsupported-authorisation-protocol)


```shell
echo $DISPLAY

# 注意：`xauth add` 之后可能需要重新连接才能生效!!
xauth add $DISPLAY . `mcookie`
xauth list

# xhost 命令并不影响最后的显示，但是可以用来测试 DISPLAY 是否可用
xhost +/-
```

# 具体方法
## Host侧配置
+ 首先需要一个支持 X11 Forwarding 功能的 Terminal，比如 MobaXterm/SecureCRT，可能是通过`ssh -X $IP`实现的
+ 通过`xauth add $DISPLAY . $(mcookie)`配置`~/.Xauthority`文件 (如果不存在，先`touch ~/.Xauthority`)
+ 运行`gvim`测试效果 (通过`sudo apt install -y vim-gtk3`安装`gvim`)

## 容器侧配置
[How to Run GUI Applications in a Docker Container](https://www.howtogeek.com/devops/how-to-run-gui-applications-in-a-docker-container/) -- 这个链接介绍了如何使用 docker-compose.yml 简化容器启动时的配置  
[Docker运行带UI界面的应用，并将它的界面投射到你的Windows电脑-云社区-华为云](https://bbs.huaweicloud.com/blogs/281862)

### 方式一
这里使用了主机网络，传递了环境变量，映射了`/tmp/.X11-unix`目录，映射了`home`目录。  
映射`home`目录主要是为了映射`~/.Xauthority`文件，[docker最好不要映射单个文件](https://yuansmin.github.io/2019/docker-mount-single-file/)。但是这样的缺点是映射了`home`目录下的所有配置，包括`.bashrc`文件。
```shell
docker run -it --net=host -v /tmp/.X11-unix:/tmp/.X11-unix -v /home/ubuntu:/home/firefly -e DISPLAY=$DISPLAY -e GDK_SCALE -e GDK_DPI_SCALE ${IMAGE_ID} /bin/bash
```

### 方式二
与方式一不同的地方是没有映射`home`目录，所有需要手动编辑`~/.Xauthority`文件。
`xauth add` 时最后一个参数需要在host上使用 `xauth list` 查询，必须完全一致才行。
```shell
ubuntu@VM-16-4-ubuntu:~ $ docker run -it --net=host -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY -e GDK_SCALE -e GDK_DPI_SCALE ${IMAGE_ID} /bin/bash
firefly@VM-16-4-ubuntu:/ $ touch ~/.Xauthority
firefly@VM-16-4-ubuntu:/ $ xauth add $DISPLAY MIT-MAGIC-COOKIE-1 d585de74d6c255c78998f7345e2ee72b
```