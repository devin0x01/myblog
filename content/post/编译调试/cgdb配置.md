---
title: "cgdb配置"
date: 2023-06-18T14:05:25+08:00
tags: ["编译调试"]
categories: []
draft: false
---
## 1.安装
```c
git clone git://github.com/cgdb/cgdb.git
cd cgdb
./autogen.sh
./configure --prefix=/usr/local
make
sudo make install

# 报错解决方法
sudo apt-get install automake libncurses5-dev flex texinfo libreadline-dev
```

## 2.配置
```c
vim ~/.cgdb/cgdbrc

set ignorecaseset
ts=4
set wso=vertical
set eld=shortarrow
set hls
map <F9> :until<cr>
```
## 3.快捷键
> F5 - Send a run command to GDB.
> F6 - Send a continue command to GDB.
> F7 - Send a finish command to GDB.
> F8 - Send a next command to GDB.
> F10 - Send a step command to GDB.
