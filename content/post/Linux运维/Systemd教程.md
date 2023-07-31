---
title: "Systemd教程"
date: 2023-07-31T16:05:25+08:00
tags: ["Linux运维"]
categories: []
draft: false
---

[Systemd 入门教程：实战篇 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)

# 配置文件
```shell
$ systemctl cat sshd.service

[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

## Unit区块
`Unit`区块：启动顺序与依赖关系。  
`Description`字段给出当前服务的简单描述。  
`Documentation`字段给出文档位置。  
`After`字段表示如果`network.target`或`sshd-keygen.service`需要启动，那么`sshd.service`应该在它们之后启动。  
`Before`字段定义`sshd.service`应该在哪些服务之前启动。  
**注意，After和Before字段只涉及启动顺序，不涉及依赖关系。**

设置依赖关系，需要使用`Wants`字段和`Requires`字段。
`Wants`字段表示`sshd.service`与`sshd-keygen.service`之间存在"弱依赖"关系，即如果`sshd-keygen.service`启动失败或停止运行，不影响sshd.service继续执行。
`Requires`字段则表示"强依赖"关系，即如果该服务启动失败或异常退出，那么`sshd.service`也必须退出。
**注意，Wants字段与Requires字段只涉及依赖关系，与启动顺序无关，默认情况下是同时启动的。**

举例来说，某 Web 应用需要 postgresql 数据库储存数据。在配置文件中，它只定义要在 postgresql 之后启动，而没有定义依赖 postgresql 。上线后，由于某种原因，postgresql 需要重新启动，在停止服务期间，该 Web 应用就会无法建立数据库连接。

## Service区块
`Service`区块：启动行为，Service区块定义如何启动当前服务。

`EnvironmentFile`字段指定当前服务的环境参数文件。该文件内部的key=value键值对，可以用$key的形式，在当前配置文件中获取。上面的例子中，sshd 的环境参数文件是`/etc/sysconfig/sshd`。  
配置文件里面最重要的字段是`ExecStart`。`ExecStart`字段：定义启动进程时执行的命令。  
上面的例子中，启动sshd，执行的命令是`/usr/sbin/sshd -D $OPTIONS`，其中的变量`$OPTIONS`就来自`EnvironmentFile`字段指定的环境参数文件。

## Install区块
`Install`区块，定义如何安装这个配置文件，即怎样做到开机启动。  
`WantedBy`字段：表示该服务所在的 `Target`。