## 目录

- [UNIX 域协议特点](#UNIX-域协议特点)
- [UNIX 域地址结构](#UNIX-域地址结构)
- [UNIX 域字节流回射服务客户端程序](#UNIX-域字节流回射服务客户端程序)
- [UNIX 域套接字编程注意点](#UNIX-域套接字编程注意点)

## UNIX 域协议特点

- UNIX 域套接字与 TCP 相比, 在同一台主机上, UNIX 域套接字更有效率, 几乎是 TCP 的两倍(由于 UNIX 域套接字不需要经过网络协议栈,不需要打包/拆包,计算校验和,维护序号和应答等,只是将应用层数据从一个进程拷贝到另一个进程, 而且 UNIX 域协议机制本质上就是可靠的通讯，而网络协议是为不可靠的通讯设计的)。
- UNIX 域套接字可以在同一台主机上各进程之间传递文件描述符。
- UNIX 域套接字与传统套接字的区别是用路径名来表示协议族的描述。

UNIX 域套接字也提供面向流和面向数据包两种 API 接口，类似于 TCP 和 UDP，但是面向消息的 UNIX 套接字也是可靠的,消息既不会丢失也不会顺序错乱。

使用 UNIX 域套接字的过程和网络 socket 十分相似, 也要先调用 socket 创建一个 socket 文件描述符, family 指定为 AF_UNIX, type 可以选择
 SOCK_DGRAM/SOCK_STREAM。

## UNIX 域地址结构

```c
#define UNIX_PATH_MAX    108
struct sockaddr_un
{
    sa_family_t sun_family;               /* AF_UNIX */
    char        sun_path[UNIX_PATH_MAX];  /* pathname */
};
```

##  UNIX 域字节流回射服务客户端程序

```c++
// server
#include <iostream>
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdlib.h>

using namespace std;

#define ERR_EXIT(m) \
        do \
        { \
            perror(m); \
            exit(EXIT_FAILURE); \
        } while(0);


void echosrv(int clientfd)
{
    char recvbuf[1024];
    while (1) {
        memset(recvbuf, 0, sizeof recvbuf);
        int ret;
        if ((ret = read(clientfd, recvbuf, sizeof(recvbuf))) < 0) {
            ERR_EXIT("read");
        }
        if (ret == 0) {
            printf("client close\n");
            break;
        }
        fputs(recvbuf, stdout);
        write(clientfd, &recvbuf, strlen(recvbuf));
    }
    close(clientfd);
}

int main(int argc, char** argv)
{
    int listenfd;

    unlink("/home/jxq/CLionProjects/NetworkProgramming/text_unix");    // 解除原有text_unix对象链接
    if ((listenfd = socket(AF_UNIX, SOCK_STREAM, 0)) < 0)
    {
        ERR_EXIT("socket");
    }

    struct sockaddr_un servaddr;
    servaddr.sun_family = AF_UNIX;
    strcpy(servaddr.sun_path, "/home/jxq/CLionProjects/NetworkProgramming/text_unix");

    if (bind(listenfd, (struct sockaddr*)&servaddr, sizeof servaddr) < 0)
    {
        ERR_EXIT("bind");
    }

    if (listen(listenfd, SOMAXCONN) < 0)
    {
        ERR_EXIT("listen");
    }

    struct sockaddr_un cliaddr;
    socklen_t clilen;
    int clientfd;
    while (1)
    {
        if ((clientfd = accept(listenfd, (struct sockaddr*)&cliaddr, &clilen)) < 0)
        {
            ERR_EXIT("accept");
        }
        echosrv(clientfd);
    }
    close(listenfd);

    return 0;
}
```

```c++
// client
#include <iostream>
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdlib.h>

using namespace std;

#define ERR_EXIT(m) \
        do \
        { \
            perror(m); \
            exit(EXIT_FAILURE); \
        } while(0);


void clisrv(int sock)
{
    char sendbuf[1024];
    char recvbuf[1024];
    memset(recvbuf, 0, sizeof(recvbuf));
    memset(sendbuf, 0, sizeof(sendbuf));
    while (fgets(sendbuf, sizeof sendbuf, stdin) != NULL)
    {
        if (write(sock, sendbuf, sizeof(sendbuf)) < 0)
        {
            ERR_EXIT("write");
        }
        int ret;
        if ((ret = read(sock, &recvbuf, sizeof recvbuf)) < 0)
        {
            ERR_EXIT("read");
        }
        if (ret == 0)
        {
            printf("server close\n");
            break;
        }
        fputs(recvbuf, stdout);
        memset(recvbuf, 0, sizeof(recvbuf));
        memset(sendbuf, 0, sizeof(sendbuf));
    }
}


int main(int argc, char** argv)
{
    int clientfd;
    if ((clientfd = socket(AF_UNIX, SOCK_STREAM, 0)) < 0)
    {
        ERR_EXIT("sokcet");
    }

    struct sockaddr_un cliaddr;
    cliaddr.sun_family = AF_UNIX;
    strcpy(cliaddr.sun_path, "/home/jxq/CLionProjects/NetworkProgramming/text_unix");

    int conn;
    if ((conn = connect(clientfd, (struct sockaddr*)&cliaddr, sizeof(cliaddr))) < 0)
    {
        ERR_EXIT("connect");
    }

    clisrv(clientfd);


    return 0;
}
```

## UNIX 域套接字编程注意点

1. bind 成功将会创建一个文件，权限为 0777 & ~umask
2. sun_path 最好用一个 /tmp 目录下的文件的绝对路径, 而且 server 端在指定该文件之前首先要 unlink 一下
3. UNIX 域协议支持流式套接口(需要处理粘包问题)与报式套接口(基于数据报)
4. UNIX 域流式套接字 connect 发现监听队列满时，会立刻返回一个 ECONNREFUSED，这和 TCP 不同，如果监听队列满，会忽略到来的 SYN，这导致对方重传 SYN










