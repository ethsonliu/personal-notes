# 目录

- [read&write 与 recv&send 区别](#read&write-与-recv&send-区别)
- [getsockname 和 getpeername](#getsockname-和-getpeername)
- [gethostname 和 gethostbyname 和 gethostbyaddr](#gethostname-和-gethostbyname-和-gethostbyaddr)
- [gethostbyaddr](#gethostbyaddr)
- [getaddrinfo](#getaddrinfo)

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

# gethostname 和 gethostbyname

```c
// 返回本地主机的标准主机名。

#include <unistd.h>
int gethostname(char *name, size_t len); // 不适用于多线程同时调用，建议使用下面讲解的 getaddrinfo
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

# gethostbyaddr

```c
// 与 gethostbyname 相反，这个是 ip 转域名
#include <sys/socket.h>       /* for AF_INET */
struct hostent *gethostbyaddr(const void *addr, socklen_t len, int type);
```

```c

// 仅供参考，可能有错

#include <netdb.h>
#include <sys/socket.h>

int main(int argc, char **argv)
{
    char *ptr, **pptr;
    struct hostent *hptr;
    char str[32];
    char ipaddr[16];
    struct in_addr *hipaddr;
    /* 取得命令后第一个参数，即要解析的IP地址 */
    ptr = argv[1];
     /* 调用inet_aton()，ptr就是以字符串存放的地方的指针，hipaddr是in_addr形式的地址 */
    if (!inet_aton(ptr, hipaddr))
    {
        printf("inet_aton error/n");
        return 1;
    }
     /* 调用gethostbyaddr()。调用结果都存在hptr中 */
    if ((hptr = gethostbyaddr(hipaddr, 4, AF_INET)) == NULL)
    {
        printf("gethostbyaddr error for addr:%s/n", ptr);
        return 1; /* 如果调用gethostbyaddr发生错误，返回1 */
    }

    /* 将主机的规范名打出来 */
    printf("official hostname:%s/n", hptr->h_name);
    /* 主机可能有多个别名，将所有别名分别打出来 */
    for (pptr = hptr->h_aliases; *pptr != NULL; pptr++)
        printf("  alias:%s/n", *pptr);

    /* 根据地址类型，将地址打出来 */
    switch (hptr->h_addrtype)
    {
        case AF_INET :
        case AF_INET6 :
             pptr = hptr->h_addr_list;
             /* 将刚才得到的所有地址都打出来。其中调用了inet_ntop()函数 */
             for (; *pptr != NULL; pptr++)
                 printf("  address:%s/n", inet_ntop(hptr->h_addrtype, *pptr, str, sizeof(str)));
             break;
        default :
            printf("unknown address type/n");
            break;
    }
    return 0;
}
```

# getaddrinfo

getaddrinfo() + addrinfo 是一对更现代、方便的组合，用于取代 gethostbyname() + sockaddr_in。

不仅可以做 DNS lookups，也可以做 services name lookups。二合一。

思路为，传入服务器的地址和端口，外加一个 addrinfo（用于描述服务器），就可以得到一个更具体的 addrinfo，这个结果可以在创建 socket 时使用。

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *node,     // e.g. "www.example.com" or IP
                const char *service,  // e.g. "http" or port number
                const struct addrinfo *hints, //指定一些基本值
                struct addrinfo **res);//得到链表
```

```c
#include <netdb.h>

struct addrinfo {
    int              ai_flags; // 常用的值是 AIPASSIVE。与 node 为 NULL 作为组合，用来做服务端。（passive 就是被动、被动接入的意思）
    int              ai_family; // 常用的值是 AFUNSPEC，表示 ipv4，ipv6 皆可
    int              ai_socktype; //SOCK_STREAM SOCK_DGRAM
    int              ai_protocol; //TCP UDP...
    socklen_t        ai_addrlen;
    struct sockaddr *ai_addr;
    char            *ai_canonname;
    struct addrinfo *ai_next;
};
```

例子，

```c
#define PORT "3490"
struct addrinfo hints;

//Step1. 指定选项
memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // use my IP

//Step2. 调用getaddrinfo
if ((rv = getaddrinfo(NULL, PORT, &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
    return 1;
}

//Step3. Use the result
// loop through all the results and bind to the first we can
for(p = servinfo; p != NULL; p = p->ai_next) {
    if ((sockfd = socket(p->ai_family, p->ai_socktype,
            p->ai_protocol)) == -1) {
        perror("server: socket");
        continue;
    }

    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &yes,
            sizeof(int)) == -1) {
        perror("setsockopt");
        exit(1);
    }

    if (bind(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
        close(sockfd);
        perror("server: bind");
        continue;
    }

    break;
}

if (p == NULL)  {
    fprintf(stderr, "server: failed to bind\n");
    return 2;//不需要freeaddrinfo()
}

//Step4. Be tidy。释放得到的serverinfo。
//注意该语句位置在return 之后
//说明，只有成功时，才会分配内存（需要释放）
freeaddrinfo(servinfo); // all done with this structure
```
可以参考 MSDN 上的完整例子[server](https://docs.microsoft.com/zh-cn/windows/win32/winsock/complete-server-code?redirectedfrom=MSDN)
 和 [client](https://docs.microsoft.com/zh-cn/windows/win32/winsock/complete-client-code?redirectedfrom=MSDN)。

















