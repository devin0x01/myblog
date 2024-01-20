---
title: "Linux网络编程"
date: 2023-06-18T14:05:25+08:00
tags: ["Linux开发"]
categories: []
draft: false
toc: true
---
## 查看端口占用情况

> **netstat -tunlp**
>
> -t (tcp) 仅显示tcp相关选项     -u (udp) 仅显示udp相关选项
>
> -n 拒绝显示列名，能显示数字的全部转化为数字
>
> -l 仅显示出在listen(监听）的服务状态
>
> -p 显示潜力相关链接的程序名

[linux查看端口被哪个进程占用的方法](https://www.yisu.com/zixun/311818.html)

## 本机地址

> **127.0.0.1**: 这个地址通常分配给loopback接口，loopback是一个特殊的网络IP，可以理解为虚拟网卡，用于本机中各个应用之间的网络交互，只要操作系统网络组建正常，loopback就能工作。所以这里需要明白使用127.0.0.1进行通信时，必须要保证client和server在**同一台机器上**。
>
> **INADDR_ANY**: 从字面上的any可以看出，转换过来其实是0.0.0.0，泛指本机的意思，也就是标识本机的所有IP，因为有些机器是不止一块网卡的，在多网卡的情况下，这个就表示**所有的网卡IP地址**的意思，不管数据是从哪个网卡过来的，只要是绑定的端口号过来的数据，都可以接收到。

如果现在有两台PC在同一个局域网内，分别为PC1与PC2，PC1上有一个网卡，IP地址为`192.168.10.128`

- PC1中sever监听`127.0.0.1`，则PC1中的client可以连上`127.0.0.1`，`192.168.10.128`连不上；而PC2中client都连不上。
- PC1中sever监听`192.168.10.128`，则PC1中的client可以连上`192.168.10.128`，`127.0.0.1`连不上；而PC2中client能连上`192.168.10.128`。
- PC1中sever监听`0.0.0.0`，则PC1中的client可以连上`127.0.0.1`和`192.168.10.128`，PC2中的client能连上`192.168.10.128`。

## socket基础接口

### close关闭socket

调用`close`函数，会向连接的对应套接字发送`EOF` (*ch04/echo_server.c*)

> **文件结束符**：如果 read()调用成功，将返回实际读取的字节数，如果遇到文件结束（EOF）则返回 0，如果出现错误则返回-1。ssize_t 数据类型属于有符号的整数类型，用来存放（读取的）字节数或-1（表示错误）。 
>
> **终端特殊字符**：EOF 是传统模式下的文件结尾字符（通常是 **Ctrl-D**）。在一行的开始处输入这个字符会导致在终端上读取输入的进程检测到文件结尾的情况（即，read()返回 0）。如果不在一行的开始处，而在其他地方输入这个字符，那么该字符会立刻导致 read()完成调用，返回这一行中目前为止读取到的字符数。在这两种情况下，EOF 字符本身都不会传递给读取的进程。

![](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_bb12f6be8f2d72b768f70889cd3d83c8.png)

### shutdown优雅的断开TCP连接

```c
/* Shut down all or part of the connection open on socket FD.
   HOW determines what to shut down:
     SHUT_RD   = No more receptions;
     SHUT_WR   = No more transmissions;
     SHUT_RDWR = No more receptions or transmissions.
   Returns 0 on success, -1 for errors.  */
extern int shutdown (int __fd, int __how) __THROW;
```


## IO复用

### 1.select接口
```cpp
// echo_client.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 1024
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len;
    struct sockaddr_in serv_adr;

    if (argc != 3)
    {
        printf("Usage : %s <IP> <port>\n", argv[0]);
        exit(1);
    }

    sock = socket(PF_INET, SOCK_STREAM, 0);
    if (sock == -1)
        error_handling("socket() error");

    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_adr.sin_port = htons(atoi(argv[2]));

    if (connect(sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
        error_handling("connect() error!");
    else
        puts("Connected...........");

    while (1)
    {
        fputs("Input message(Q to quit): ", stdout);
        fgets(message, BUF_SIZE, stdin);

        if (!strcmp(message, "q\n") || !strcmp(message, "Q\n"))
            break;

        write(sock, message, strlen(message));
        str_len = read(sock, message, BUF_SIZE - 1);
        message[str_len] = 0;
        printf("Message from server: %s", message);
    }
    close(sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```
```cpp
// echo_selectserv.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/time.h>
#include <sys/select.h>

#define BUF_SIZE 100
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int serv_sock, clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    struct timeval timeout;
    fd_set reads, cpy_reads;

    socklen_t adr_sz;
    int fd_max, str_len, fd_num, i;
    char buf[BUF_SIZE];
    if (argc != 2)
    {
        printf("Usage : %s <port>\n", argv[0]);
        exit(1);
    }
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    if (bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
        error_handling("bind() error");
    if (listen(serv_sock, 5) == -1)
        error_handling("listen() error");

    FD_ZERO(&reads);
    FD_SET(serv_sock, &reads); //注册服务端套接字
    fd_max = serv_sock;

    while (1)
    {
        cpy_reads = reads;
        timeout.tv_sec = 5;
        timeout.tv_usec = 5000;

        if ((fd_num = select(fd_max + 1, &cpy_reads, 0, 0, &timeout)) == -1) //开始监视,每次重新监听
            break;
        if (fd_num == 0)
            continue;

        for (i = 0; i < fd_max + 1; i++)
        {
            if (FD_ISSET(i, &cpy_reads)) //查找发生变化的套接字文件描述符
            {
                if (i == serv_sock) //如果是服务端套接字时,受理连接请求
                {
                    adr_sz = sizeof(clnt_adr);
                    clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_adr, &adr_sz);

                    FD_SET(clnt_sock, &reads); //注册一个clnt_sock
                    if (fd_max < clnt_sock)
                        fd_max = clnt_sock;
                    printf("Connected client: %d \n", clnt_sock);
                }
                else //不是服务端套接字时
                {
                    str_len = read(i, buf, BUF_SIZE); //i指的是当前发起请求的客户端
                    if (str_len == 0)
                    {
                        FD_CLR(i, &reads);
                        close(i);
                        printf("closed client: %d \n", i);
                    }
                    else
                    {
                        write(i, buf, str_len);
                    }
                }
            }
        }
    }
    close(serv_sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

```cpp
extern int select(int maxfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds,
                  struct timeval *__timeout);
```

![](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_122999c4ec9f2f1b4a88eee6165a9de4.png)

### 2.epoll系列接口
```cpp
// 水平触发 echo_EPLTserv.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>

#define BUF_SIZE 2
#define EPOLL_SIZE 50
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int serv_sock, clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    socklen_t adr_sz;
    int str_len, i;
    char buf[BUF_SIZE];

    struct epoll_event *ep_events;
    struct epoll_event event;
    int epfd, event_cnt;

    if (argc != 2)
    {
        printf("Usage : %s <port> \n", argv[0]);
        exit(1);
    }
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    if (bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
        error_handling("bind() error");
    if (listen(serv_sock, 5) == -1)
        error_handling("listen() error");

    epfd = epoll_create(EPOLL_SIZE); //可以忽略这个参数，填入的参数为操作系统参考
    ep_events = malloc(sizeof(struct epoll_event) * EPOLL_SIZE);

    event.events = EPOLLIN; //需要读取数据的情况
    event.data.fd = serv_sock;
    epoll_ctl(epfd, EPOLL_CTL_ADD, serv_sock, &event); //例程epfd 中添加文件描述符 serv_sock，目的是监听 enevt 中的事件

    while (1)
    {
        event_cnt = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1); //获取改变了的文件描述符，返回数量
        if (event_cnt == -1)
        {
            puts("epoll_wait() error");
            break;
        }

        puts("return epoll_wait");
        for (i = 0; i < event_cnt; i++)
        {
            if (ep_events[i].data.fd == serv_sock) //客户端请求连接时
            {
                adr_sz = sizeof(clnt_adr);
                clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_adr, &adr_sz);
                event.events = EPOLLIN;
                event.data.fd = clnt_sock; //把客户端套接字添加进去
                epoll_ctl(epfd, EPOLL_CTL_ADD, clnt_sock, &event);
                printf("connected client : %d \n", clnt_sock);
            }
            else //是客户端套接字时
            {
                str_len = read(ep_events[i].data.fd, buf, BUF_SIZE);
                if (str_len == 0)
                {
                    epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL); //从epoll中删除套接字
                    close(ep_events[i].data.fd);
                    printf("closed client : %d \n", ep_events[i].data.fd);
                }
                else
                {
                    write(ep_events[i].data.fd, buf, str_len);
                }
            }
        }
    }
    close(serv_sock);
    close(epfd);

    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```
```cpp
// 边沿触发 echo_EPETserv.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <errno.h>

#define BUF_SIZE 4 //缓冲区设置为 4 字节
#define EPOLL_SIZE 50
void setnonblockingmode(int fd);
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int serv_sock, clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    socklen_t adr_sz;
    int str_len, i;
    char buf[BUF_SIZE];

    struct epoll_event *ep_events;
    struct epoll_event event;
    int epfd, event_cnt;

    if (argc != 2)
    {
        printf("Usage : %s <port> \n", argv[0]);
        exit(1);
    }
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    if (bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
        error_handling("bind() error");
    if (listen(serv_sock, 5) == -1)
        error_handling("listen() error");

    epfd = epoll_create(EPOLL_SIZE); //可以忽略这个参数，填入的参数为操作系统参考
    ep_events = malloc(sizeof(struct epoll_event) * EPOLL_SIZE);

    setnonblockingmode(serv_sock);
    event.events = EPOLLIN; //需要读取数据的情况
    event.data.fd = serv_sock;
    epoll_ctl(epfd, EPOLL_CTL_ADD, serv_sock, &event); //例程epfd 中添加文件描述符 serv_sock，目的是监听 enevt 中的事件

    while (1)
    {
        event_cnt = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1); //获取改变了的文件描述符，返回数量
        if (event_cnt == -1)
        {
            puts("epoll_wait() error");
            break;
        }

        puts("return epoll_wait");
        for (i = 0; i < event_cnt; i++)
        {
            if (ep_events[i].data.fd == serv_sock) //客户端请求连接时
            {
                adr_sz = sizeof(clnt_adr);
                clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_adr, &adr_sz);
                setnonblockingmode(clnt_sock);    //将 accept 创建的套接字改为非阻塞模式
                event.events = EPOLLIN | EPOLLET; //改成边缘触发
                event.data.fd = clnt_sock;        //把客户端套接字添加进去
                epoll_ctl(epfd, EPOLL_CTL_ADD, clnt_sock, &event);
                printf("connected client : %d \n", clnt_sock);
            }
            else //是客户端套接字时
            {
                while (1)
                {
                    str_len = read(ep_events[i].data.fd, buf, BUF_SIZE);
                    if (str_len == 0)
                    {
                        epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL); //从epoll中删除套接字
                        close(ep_events[i].data.fd);
                        printf("closed client : %d \n", ep_events[i].data.fd);
                        break;
                    }
                    else if (str_len < 0)
                    {
                        if (errno == EAGAIN) //read 返回-1 且 errno 值为 EAGAIN ，意味读取了输入缓冲的全部数据
                            break;
                    }
                    else
                    {
                        write(ep_events[i].data.fd, buf, str_len);
                    }
                }
            }
        }
    }
    close(serv_sock);
    close(epfd);

    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
void setnonblockingmode(int fd)
{
    int flag = fcntl(fd, F_GETFL, 0);
    fcntl(fd, F_SETFL, flag | O_NONBLOCK);
}
```

```cpp
// size: epoll实例监听的文件描述符数目。Linux2.6.8之后这个参数可以忽略。
// retval: epoll实例文件描述符
int epoll_create(int size);

// op: EPOLL_CTL_ADD, EPOLL_CTL_DEL, EPOLL_CTL_MOD
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；

// events: 保存发生事件的文件描述符
// maxevents: 第二个参数可以保存的最大事件数，即数组大小
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);


typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

// events支持的事件类型：
// EPOLLIN: 需要读取数据的情况
// EPOLLOUT: 输出缓存为空，可以理解发送数据的情况
// EPOLLPRI: 收到OOB数据的情况
// EPOLLRDHUP: 断开连接或半关闭的情况，这在边缘触发方式下非常有用
// EPOLLERR: 发生错误
// EPOLLET: 以边沿触发的方式得到事件通知
// EPOLLONESHOT: 发生一次事件后，相应文件描述符不再收到事件通知。因此需要先epoll_ctl函数的第2个参数传递EPOLL_CTL_MOD再次设置事件。
struct epoll_event
{
  uint32_t events;	/* Epoll events */
  epoll_data_t data;	/* User data variable */
};
```

> 1.为什么不可以同时设置`event.events = EPOLLIN | EPOLLOUT`？
>
> 最主要的是这句：at the time of the callback, epoll has no clue of what happened。大概指的回调的时候，`epoll`是不能区分出是`EPOLLIN`还是`EPOLLOUT`的。换句话说如果你同时监听了`EPOLLIN`和`EPOLLOUT`，当发生了`EPOLLIN`，由于`epoll`无法区分种类，那么将会同时监听到`EPOLLIN`和`EPOLLOUT`事件（即使当时没有发生EPOLLOUT）。
>
> 2.什么时候设置为`EPOLLOUT`呢？
>
> 当需要发送数据的时候，可以使用`epoll_ctl`来注册事件，第3个参数可以设置为`event->events = EPOLLOUT; event->data.ptr = dataToBeSent`，其中`dataToBeSent`可以为自定义结构`struct { int fd; char *buf; }`。当发生`EPOLLOUT`时候表示可以继续写入，直到发送完毕，然后再设置为`EPOLLIN`。
>
> [EPOLL讲解、注意点和使用建议 - 简书 (jianshu.com)](https://www.jianshu.com/p/f1eae48c1211)



**epoll默认是水平触发模式。边沿触发ET相比于水平触发LT，改动的地方主要有2处：**

- 设置为fd为非阻塞IO
- 读取时需要使用while(1)读取，读取完成后read返回-1而且errno==EAGAIN

![](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_90475ad986a2c3183867e607e0799f3c.png)

### 3.select和epoll的比较

> 1.select相比epoll的速度慢的原因：
>
> - 调用select函数后需要遍历所有文件描述符
>
> - 每次调用select函数时，都需要向该函数传递监视对象信息。
>
> 2.select相比epoll的**优势**：
>
> - epoll只支持Linux平台，而select支持大部分操作系统。因此服务端接入这比较少时适用。
