## 目录

- [UNIX 域协议特点](#UNIX-域协议特点)
- [UNIX 域地址结构](#UNIX-域地址结构)
- [UNIX 域字节流回射服务客户端程序](#UNIX-域字节流回射服务客户端程序)
- [UNIX 域套接字编程注意点](#UNIX-域套接字编程注意点)
- [socketpair](#socketpair)
- [sendmsg & recvmsg](#sendmsg-&-recvmsg)
- [UNIX 域套接字传递描述符字](#UNIX-域套接字传递描述符字)

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

## socketpair

```c
#include <sys/types.h>
#include <sys/socket.h>

int socketpair(int domain, int type, int protocol, int sv[2]);
```

创建一个全双工的流管道。

- domain: 协议家族, 可以使用 AF_UNIX(AF_LOCAL)UNIX 域协议, 而且在 Linux 上, 该函数也就只支持这一种协议
- type: 套接字类型, 可以使用 SOCK_STREAM
- protocol: 协议类型, 一般填充为 0
- sv: 返回的套接字对
- 返回值：成功返回 0，失败返回 -1

socketpair 函数跟 pipe 函数是类似: 只能在具有亲缘关系的进程间通信，但 pipe 创建的匿名管道是半双工的，而 socketpair ：可以认为是创建一个全双工的管道。

可以使用 socketpair 创建返回的套接字对进行父子进程通信, 如下例:

```c
int main()
{
    int sockfds[2];
    if (socketpair(AF_UNIX, SOCK_STREAM, 0, sockfds) == -1)
        err_exit("socketpair error");
 
    pid_t pid = fork();
    if (pid == -1)
        err_exit("fork error");
    // 父进程, 只负责数据的打印
    else if (pid > 0)
    {
        close(sockfds[1]);
        int iVal = 0;
        while (true)
        {
            cout << "value = " << iVal << endl;
            write(sockfds[0], &iVal, sizeof(iVal));
            read(sockfds[0], &iVal, sizeof(iVal));
            sleep(1);
        }
    }
    // 子进程, 只负责数据的更改(+1)
    else if (pid == 0)
    {
        close(sockfds[0]);
        int iVal = 0;
        while (read(sockfds[1], &iVal, sizeof(iVal)) > 0)
        {
            ++ iVal;
            write(sockfds[1], &iVal, sizeof(iVal));
        }
    }
}
```

## sendmsg & recvmsg

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

它们与 sendto/send 和 recvfrom/recv 函数类似,只不过可以传输更复杂的数据结构，不仅可以传输一般数据，还可以传输额外的数据，如文件描述符。

```c
struct msghdr
{
    void         *msg_name;       /* optional address */
    socklen_t     msg_namelen;    /* size of address */
    struct iovec *msg_iov;        /* scatter/gather array */
    size_t        msg_iovlen;     /* # elements in msg_iov */
    void         *msg_control;    /* ancillary data, see below */
    size_t        msg_controllen; /* ancillary data buffer len */
    int           msg_flags;      /* flags on received message */
};

struct iovec                      /* Scatter/gather array items */
{
    void  *iov_base;              /* Starting address */
    size_t iov_len;               /* Number of bytes to transfer */
};
```

msghdr 结构体成员解释:

- msg_name：即对等方的地址指针，不关心时设为 NULL 即可
- msg_namelen：地址长度，不关心时设置为 0 即可
- msg_iov：是结构体 iovec 的指针, 指向需要发送的普通数据
- msg_iovlen：当有 n 个 iovec 结构体时，此值为 n
- msg_control：是一个指向 cmsghdr 结构体的指针, 当需要发送辅助数据(如控制信息/文件描述符)时, 需要设置该字段, 当发送正常数据时, 就不需要关心该字段, 并且 msg_controllen 可以置为 0
- msg_controllen：cmsghdr 结构体可能不止一个
- msg_flags: 不用关心
- iov_base：可以认为是传输正常数据时的 buf
- iov_len：buf 的大小

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/023.jpg)

```c

struct cmsghdr
{
    socklen_t cmsg_len;    /* data byte count, including header */
    int       cmsg_level;  /* originating protocol */
    int       cmsg_type;   /* protocol-specific type */
    /* followed by unsigned char cmsg_data[]; */
};
```

为了对齐，可能存在一些填充字节(见下图)，跟系统的实现有关，但我们不必关心，可以通过一些函数宏来获取相关的值，如下：

```c
#include <sys/socket.h>

struct cmsghdr *CMSG_FIRSTHDR(struct msghdr *msgh);	// 获取辅助数据的第一条消息
struct cmsghdr *CMSG_NXTHDR(struct msghdr *msgh, struct cmsghdr *cmsg);	// 获取辅助数据的下一条信息
size_t CMSG_ALIGN(size_t length);	
size_t CMSG_SPACE(size_t length);
size_t CMSG_LEN(size_t length);	// length使用的是的(实际)数据的长度, 见下图(两条填充数据的中间部分)
unsigned char *CMSG_DATA(struct cmsghdr *cmsg);
```

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/024.png)

## UNIX 域套接字传递描述符字
