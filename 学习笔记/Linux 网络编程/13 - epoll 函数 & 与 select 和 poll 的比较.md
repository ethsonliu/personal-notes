## 目录

- [epoll 介绍](#epoll-介绍)
- [示例代码](#示例代码)
- [水平触发和边沿触发](#水平触发和边沿触发)
- [select & poll & epoll 比较](#select-&-poll-&-epoll-比较)
- [参考](#参考)

## epoll 介绍

epoll 是Linux 下多路复用 IO 接口 select/poll 的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率，因为它会复用文件描述符集用来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集（内部红黑树实现），只要遍历那些被内核 IO 事件异步唤醒而加入 Ready 队列的描述符集合就行了。

epoll 除了提供 select/poll 那种 IO 事件的水平触发 （Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存 IO 状态，减少 epoll_wait/epoll_pwait 的调用，提高应用程序效率。

### epoll 实现机制

![](https://github.com/ethsonliu/personal-notes/blob/master/_image/037.webp)

在 linux，一切皆文件．所以当调用 epoll_create 时，内核给这个 epoll 分配一个 file，但是这个不是普通的文件，而是只服务于 epoll。

所以当内核初始化 epoll 时，会开辟一块内核高速 cache 区，用于安置我们监听的 socket，这些 socket 会以红黑树的形式保存在内核的 cache 里，以支持快速的查找，插入，删除．同时，建立了一个 list 链表，用于存储准备就绪的事件．所以调用 epoll_wait 时，在 timeout 时间内，只是简单的观察这个 list 链表是否有数据，如果没有，则睡眠至超时时间到返回；如果有数据，则在超时时间到，拷贝至用户态 events 数组中。

那么，这个准备就绪 list 链表是怎么维护的呢？当我们执行 epoll_ctl 时，除了把 socket 放到 epoll 文件系统里 file 对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪 list 链表里。所以，当一个 socket 上有数据到了，内核在把网卡上的数据 copy 到内核中后，就来把 socket 插入到准备就绪链表里了。

epoll 操作过程需要三个接口，分别如下：

```c
#include <sys/epoll.h>

int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

### int epoll_create(int size)

man 参考：http://man7.org/linux/man-pages/man2/epoll_create.2.html

epoll_create 建立一个 epoll 对象。参数 size 是内核保证能够正确处理的最大句柄数，多于这个最大数时内核可不保证效果。但这个参数从 Linux 2.6.8 起就被忽略了，只需大于 0 即可。

当创建好 epoll 句柄后，它就是会占用一个 fd 值，在 linux 下如果查看 /proc/id/fd/，是能够看到这个 fd 的，所以在使用完 epoll 后，必须调用 close() 关闭，否则可能导致 fd 被耗尽。

### int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)

man 参考：http://man7.org/linux/man-pages/man2/epoll_ctl.2.html

epoll 的事件注册函数，它不同于 select() 是在监听事件时告诉内核要监听什么类型的事件， epoll 是在这里先注册要监听的事件类型。

第一个参数是 epoll_create() 的返回值，第二个参数表示动作，用三个宏来表示：

```
EPOLL_CTL_ADD // 注册新的 fd 到 epfd 中
EPOLL_CTL_MOD // 修改已经注册的 fd 的监听事件
EPOLL_CTL_DEL // 从 epfd 中删除一个 fd
```

第三个参数是需要监听的 fd。

第四个参数是告诉内核需要监听什么事，struct epoll_event 结构如下：

```c
typedef union epoll_data {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
}epoll_data_t;

struct epoll_event {
    uint32_t events;   /* Epoll events */
    epoll_data_t data; /* User data variable */
};
```

events 可以是以下几个宏的集合：

```
EPOLLIN      // 表示对应的文件描述符可以读（包括对端 SOCKET 正常关闭）
EPOLLOUT     // 表示对应的文件描述符可以写
EPOLLPRI     // 表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来)
EPOLLERR     // 表示对应的文件描述符发生错误
EPOLLHUP     // 表示对应的文件描述符被挂断
EPOLLET      // 将 EPOLL 设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的
EPOLLONESHOT // 只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个 socket 的话，需要再次把这个 socket 加入到 EPOLL 队列里
```

### int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout)

man 参考：http://man7.org/linux/man-pages/man2/epoll_wait.2.html

等待事件的产生，类似于 select() 调用。

参数 events 用来从内核得到事件的集合，maxevents 告之内核这个 events 有多大，这个 maxevents 的值不能大于创建 epoll_create() 时的 size。

参数 timeout 是超时时间（毫秒，0 会立即返回，-1 将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回 0 表示已超时。

epoll_wait 范围之后应该是一个循环，遍历所有的事件。

## 示例代码

```c++
#include <iostream>
#include <stdio.h>
#include <cstring>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <poll.h>
#include <vector>
#include <sys/stat.h>
#include <fcntl.h>
#include <algorithm>


using namespace std;

typedef vector<struct epoll_event> EventList;

struct packet
{
    int len;
    char buf[1024];
};

#define ERR_EXIT(m) \
        do  \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while(0);

ssize_t readn(int fd, void *buf, size_t count)
{
    size_t nleft = count;   // 剩余字节数
    ssize_t nread;
    char *bufp = (char*) buf;

    while (nleft > 0)
    {
        nread = read(fd, bufp, nleft);
        if (nread < 0)
        {
            if (errno == EINTR)
            {
                continue;
            }
            return  -1;
        } else if (nread == 0)
        {
            return count - nleft;
        }

        bufp += nread;
        nleft -= nread;
    }
    return count;
}

ssize_t writen(int fd, const void *buf, size_t count)
{
    size_t nleft = count;
    ssize_t nwritten;
    char* bufp = (char*)buf;

    while (nleft > 0)
    {
        if ((nwritten = write(fd, bufp, nleft)) < 0)
        {
            if (errno == EINTR)
            {
                continue;
            }
            return -1;
        }
        else if (nwritten == 0)
        {
            continue;
        }
        bufp += nwritten;
        nleft -= nwritten;
    }
    return count;
}

ssize_t recv_peek(int sockfd, void *buf, size_t len)
{
    while (1)
    {
        int ret = recv(sockfd, buf, len, MSG_PEEK); // 查看传入消息
        if (ret == -1 && errno == EINTR)
        {
            continue;
        }
        return ret;
    }
}

ssize_t readline(int sockfd, void *buf, size_t maxline)
{
    int ret;
    int nread;
    char *bufp = (char*)buf;    // 当前指针位置
    int nleft = maxline;
    while (1)
    {
        ret = recv_peek(sockfd, buf, nleft);
        if (ret < 0)
        {
            return ret;
        }
        else if (ret == 0)
        {
            return ret;
        }
        nread = ret;
        int i;
        for (i = 0; i < nread; i++)
        {
            if (bufp[i] == '\n')
            {
                ret = readn(sockfd, bufp, i+1);
                if (ret != i+1)
                {
                    exit(EXIT_FAILURE);
                }
                return ret;
            }
        }
        if (nread > nleft)
        {
            exit(EXIT_FAILURE);
        }
        nleft -= nread;
        ret = readn(sockfd, bufp, nread);
        if (ret != nread)
        {
            exit(EXIT_FAILURE);
        }
        bufp += nread;
    }
    return -1;
}

void echo_srv(int connfd)
{
    char recvbuf[1024];
    // struct packet recvbuf;
    int n;
    while (1)
    {
        memset(recvbuf, 0, sizeof recvbuf);
        int ret = readline(connfd, recvbuf, 1024);
        if (ret == -1)
        {
            ERR_EXIT("readline");
        }
        if (ret == 0)
        {
            printf("client close\n");
            break;
        }

        fputs(recvbuf, stdout);
        writen(connfd, recvbuf, strlen(recvbuf));
    }

}

void activate_nonblock(int fd)
{
    int ret;
    int flags = fcntl(fd, F_GETFL);
    if(flags == -1)
        ERR_EXIT("fcntl");
    flags |= O_NONBLOCK;
    ret = fcntl(fd, F_SETFL, flags);
    if(ret == -1)
        ERR_EXIT("fcntl");
}

int main(int argc, char** argv) {
    // 1. 创建套接字
    int listenfd;
    if ((listenfd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) < 0) {
        ERR_EXIT("socket");
    }

    // 2. 分配套接字地址
    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof servaddr);
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(6666);
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    // servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    // inet_aton("127.0.0.1", &servaddr.sin_addr);

    int on = 1;
    // 确保time_wait状态下同一端口仍可使用
    if (setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof on) < 0) {
        ERR_EXIT("setsockopt");
    }

    // 3. 绑定套接字地址
    if (bind(listenfd, (struct sockaddr *) &servaddr, sizeof servaddr) < 0) {
        ERR_EXIT("bind");
    }
    // 4. 等待连接请求状态
    if (listen(listenfd, SOMAXCONN) < 0) {
        ERR_EXIT("listen");
    }
    // 5. 允许连接
    struct sockaddr_in peeraddr;
    socklen_t peerlen;


    // 6. 数据交换
    int nready;
    int connfd;
    int i;
    vector<int> clients;
    int epollfd;
    epollfd = epoll_create1(EPOLL_CLOEXEC); // 创建一个多路复用的实例

    struct epoll_event event;
    event.data.fd = listenfd;
    event.events = EPOLLIN | EPOLLET;
    epoll_ctl(epollfd, EPOLL_CTL_ADD, listenfd, &event);

    EventList events(16);

    while (1)
    {
        // 等侍注册在 epfd 上的 socket fd 的事件的发生，如果发生则将发生的 sokct fd 和事件类型放入到 events 数组中
        nready = epoll_wait(epollfd, &*events.begin(),
                            static_cast<int>(events.size()), -1); // 用于轮询 I/O 事件的发生
        if (nready == -1)
        {
            if (errno == EINTR)
            {
                continue;
            }
            ERR_EXIT("epoll_wait");
        }

        if (nready == 0)
        {
            continue;
        }

        if ((size_t)nready == events.size())
        {
            events.resize(events.size()*2);
        }

        for (i = 0; i < nready; ++i)
        {
            // accept，并且将新 accept 的 fd 加进 epoll 中
            if (events[i].data.fd == listenfd) 
            {
                peerlen = sizeof(peeraddr);
                connfd = accept(listenfd, (struct sockaddr*)&peeraddr, &peerlen);
                if (connfd == -1)
                {
                    ERR_EXIT("accept");
                }
                printf("id = %s, ", inet_ntoa(peeraddr.sin_addr));
                printf("port = %d\n", ntohs(peeraddr.sin_port));
                clients.push_back(connfd);
                activate_nonblock(connfd);

                event.data.fd = connfd;
                event.events = EPOLLIN | EPOLLET;
                epoll_ctl(epollfd, EPOLL_CTL_ADD, connfd, &event);

            }
            else if (events[i].events & EPOLLIN) // 对此 fd 进行读操作
            {
                connfd = events[i].data.fd;
                if (connfd < 0)
                {
                    continue;
                }
                char recvbuf[1024];
                int ret = readline(connfd, recvbuf, sizeof(recvbuf));
                if (ret == -1)
                {
                    ERR_EXIT("readline");
                }
                if (ret == 0)
                {
                    printf("client close\n");
                    close(connfd);

                    event = events[i];
                    epoll_ctl(epollfd, EPOLL_CTL_DEL, connfd, &event);
                    clients.erase(
                            remove_if(clients.begin(), clients.end(), [connfd](int n){return n == connfd;}),
                            clients.end());
                }
                fputs(recvbuf, stdout);
                writen(connfd, recvbuf, strlen(recvbuf));
            }
            else if (events[i].events & EPOLLOUT) // 对此 fd 进行写操作
            {
            }
        }

    }
    // 7. 断开连接
    close(listenfd);
    return 0;
}
```

## 水平触发和边沿触发

### 水平触发（level triggered）

默认的工作方式，当 epoll_ wait 检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用 epoll_wait 时，会再次响应应用程序并通知此事件。

### 边沿触发（edge triggered）

当 epoll_ wait 检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用 epoll_wait 时，不会再次响应应用程序并通知此事件。

### ET 模式的原理

当一个 socket 句柄上有事件时，内核会把该句柄插入上面所说的准备就绪 list 链表，这时我们调用 epoll_ wait，会把准备就绪的 socket 拷贝到用户态内存，然后清空准备就绪 list 链表。

最后，epoll_ wait 检查这些 socket，如果不是 ET 模式（就是 LT 模式的句柄了），并且这些 socket 上确实有未处理的事件时，又把该句柄放回到刚刚清空的准备就绪链表了。

所以，非 ET 的句柄，只要它上面还有事件，epoll_ wait 每次都会返回。而 ET 模式的句柄，除非有新中断到，即使 socket 上的事件没有处理完，也是不会次次从 epoll_wait 返回的。

### 总结

ET 模式在很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。

epoll 工作在 ET 模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

## select & poll & epoll 比较

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/019.png)
![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/020.png)
![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/021.png)
![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/022.png)

## 参考

- <http://roux.top/2017/11/20/epoll%E5%87%BD%E6%95%B0/>
- <https://blog.csdn.net/lixungogogo/article/details/52226479>
