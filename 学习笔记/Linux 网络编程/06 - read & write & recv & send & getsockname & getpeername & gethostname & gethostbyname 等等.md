# 目录

- [read&write 与 recv&send 区别](#read&write-与-recv&send-区别)
- [getsockname 和 getpeername](#getsockname-和-getpeername)

# read&write 与 recv&send 区别

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);

#include <sys/types.h>
#include <sys/socket.h>
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

1. read 也可以用于文件 io，recv 只能用于 socket io。
2. recv 多了一个 flag 参数。MSG_DONTROUTE，MSG_PEEK，MSG_WAITALL 等等，参考 <http://man7.org/linux/man-pages/man2/recv.2.html>。

    MSG_DONTROUTE，是 send 函数使用的标志。这个标志告诉 IP 目的主机在本地网络上面,没有必要查找表，这个标志一般用网络诊断和路由程序里面。

    MSG_PEEK，是 recv 函数的使用标志,表示只是从系统缓冲区中读取内容,而不清除系统缓冲区的内容.这样下次读的时候,仍然是一样的内容。一般在有多个进程读写数据时
可以使用这个标志。

    MSG_WAITALL，是 recv 函数的使用标志，表示等到所有的信息到达时才返回。使用这个标志的时候 recv 会一直阻塞，直到指定的条件满足，或者是发生了错误。

# getsockname 和 getpeername

getsockname 函数用于获取与某个套接字关联的本地协议地址。getpeername 函数用于获取与某个套接字关联的外地协议地址。

```c
#include<sys/socket.h>

int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);
int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);
```

对于这两个函数，如果函数调用成功，则返回0，如果调用出错，则返回-1。

使用这两个函数，我们可以通过套接字描述符来获取自己的 IP 地址和连接对端的 IP 地址。如在未调用 bind 函数的 TCP 客户端程序上，可以通过调用getsockname() 函数获取由内核赋予该连接的本地 IP 地址和本地端口号，还可以在 TCP 的服务器端 accept 成功后，通过 getpeername() 函数来获取当前连接的客户端的 IP 地址和端口号。

参考：<https://blog.csdn.net/workformywork/article/details/24554813>

# 
