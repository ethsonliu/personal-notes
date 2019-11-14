# 目录

- [read&write 与 recv&send 区别](#read&write-与-recv&send-区别)
- [getsockname 和 getpeername](#getsockname-和-getpeername)
- [gethostname 和 gethostbyname 和 gethostbyaddr](#gethostname-和-gethostbyname-和-gethostbyaddr)

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

# gethostname 和 gethostbyname 和 gethostbyaddr

```c
// 返回本地主机的标准主机名。

#include <unistd.h>
int gethostname(char *name, size_t len);
```

接收缓冲区 name，其长度必须为 len 字节或是更，存获得的主机名。

接收缓冲区 name 的最大长度。

返回值：如果函数成功，则返回 0。如果发生错误则返回 -1。错误号存放在外部变量 errno 中。

```c
// 域名或主机名获取IP地址

#include <netdb.h>
#include <sys/socket.h>

// 这个函数的传入值是域名或者主机名，例如"www.google.cn"等等。传出值，是一个 hostent 的结构。如果函数调用失败，将返回 NULL。
struct hostent *gethostbyname(const char *name);
```

返回 hostent 结构体类型指针
          
```c
struct hostent
{
    char *h_name;       /* official name of host */
    char **h_aliases;   /* alias list */
    int h_addrtype;     /* host address type */
    int h_length;       /* length of address */
    char **h_addr_list; /* list of addresses */
}
#define h_addr h_addr_list[0] /* for backward compatibility */
```

hostent->h_name，表示的是主机的规范名。
    
hostent->h_aliases，表示的是主机的别名，有的时候，有的主机可能有好几个别名，这些，其实都是为了易于用户记忆而为自己的网站多取的名字。

hostent->h_addrtype，表示的是主机 ip 地址的类型，到底是 ipv4(AF_INET)，还是 pv6(AF_INET6)。

hostent->h_length，表示的是主机 ip 地址的长度。

hostent->h_addr_lisst，表示的是主机的 ip 地址，注意，这个是以网络字节序存储的。千万不要直接用 printf 带 %s 参数来打这个东西，会有问题的哇。所以到真正需要打印出这个 IP 的话，需要调用 inet_ntop()。

```c
// 这个函数，是将类型为 af 的网络地址结构 src，转换成主机序的字符串形式，存放在长度为 cnt 的字符串中。返回指向 dst 的一个指针。
// 如果函数调用错误，返回值是 NULL。

const char *inet_ntop(int af, const void *src, char *dst, socklen_t cnt)
```

```c
#include <stdio.h>
#include <sys/socket.h>
#include <netdb.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdlib.h>

void handler(int sig){
        printf("recv a sig=%d\n",sig);
                exit(EXIT_SUCCESS);
}


#define ERR_EXIT(m) \
        do{ \
                perror(m); \
                exit(EXIT_FAILURE);\
        }while(0);

int main(void)
{
        char host[100] = {0};
         if(gethostname(host,sizeof(host)) < 0){
                ERR_EXIT("gethostname");
        }

        struct hostent *hp;
        if ((hp=gethostbyname(host)) == NULL){
                ERR_EXIT("gethostbyname");
        }

        int i = 0;
        while(hp->h_addr_list[i] != NULL)
        {
                printf("hostname: %s\n",hp->h_name);
                printf("      ip: %s\n",inet_ntoa(*(struct in_addr*)hp->h_addr_list[i]));
                i++;
        }
        return 0;
}
```

输出如下：

```
# gcc -o getinfo getinfo.c
# ./getinfo
hostname: www.server1.com
ip: 69.172.201.208
```
