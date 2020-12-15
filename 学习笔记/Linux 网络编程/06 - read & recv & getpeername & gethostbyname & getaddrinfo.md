# 目录

- [read&write 与 recv&send 区别](#read&write-与-recv&send-区别)
- [getsockname 和 getpeername](#getsockname-和-getpeername)
- [gethostbyname](#gethostbyname)
- [gethostbyname_r](#gethostbyname_r)
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

对于这两个函数，如果函数调用成功，则返回 0，如果调用出错，则返回 -1。

使用这两个函数，我们可以通过套接字描述符来获取自己的 IP 地址和连接对端的 IP 地址。如在未调用 bind 函数的 TCP 客户端程序上，可以通过调用 getsockname() 函数获取由内核赋予该连接的本地 IP 地址和本地端口号，还可以在 TCP 的服务器端 accept 成功后，通过 getpeername() 函数来获取当前连接的客户端的 IP 地址和端口号。

参考：<https://blog.csdn.net/workformywork/article/details/24554813>

# gethostbyname

```c
#include <netdb.h>
#include <sys/socket.h>

// 作用：用域名或者主机名获取地址
// 参数 name：域名（www.baidu.com）或者主机名（hapoa-virtual-machine）
// 另注：主机名就是终端输入 hostname 得到的值
struct hostent *gethostbyname(const char *name);
```

返回的 hostent 结构体类型指针，

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

![](https://github.com/ethsonliu/personal-notes/blob/master/_image/033.png)

- hostent->h_name，表示的是主机的规范名。  
- hostent->h_aliases，表示的是主机的别名，有的时候，有的主机可能有好几个别名。
- hostent->h_addrtype，表示的是主机 ip 地址的类型，到底是 ipv4(AF_INET)，还是 pv6(AF_INET6)。
- hostent->h_length，表示的是主机 ip 地址的长度。
- hostent->h_addr_list，表示的是主机的 ip 地址，注意，这个是以网络字节序存储的。不要直接用 printf 带 %s 参数来打这个东西，会有问题的。所以到真正需要打印出这个 IP 的话，需要调用 inet_ntop。

以下是示例，

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

        struct hostent *hp;
        if ((hp=gethostbyname("www.baidu.com")) == NULL){
                ERR_EXIT("gethostbyname");
        }

        int i = 0;
        while (hp->h_addr_list[i] != NULL)
        {
                printf("hostname: %s\n",hp->h_name);
                printf("      ip: %s\n",inet_ntoa(*(struct in_addr*)hp->h_addr_list[i]));
                i++;
        }
        return 0;
}
```

输出如下：

```bash
hapoa@hapoa-virtual-machine:~$ gcc -o main ./main.cpp
hapoa@hapoa-virtual-machine:~$ ./main
hostname: www.a.shifen.com
      ip: 36.152.44.96
hostname: www.a.shifen.com
      ip: 36.152.44.95
```

参考：

- <https://blog.csdn.net/daiyudong2020/article/details/51946080>

# gethostbyname_r

因为 gethostbyname 返回的是一个指向静态变量的指针，不可重入，很可能刚要读时值就被其它线程修改，所以新的 posix 中增加了另一个可重入的函数 gethostbyname_r。（在使用前需要看看所使用系统是否有这个函数）

```c
// 参数 name：域名（www.baidu.com）或者主机名（hapoa-virtual-machine）
// 参数 ret：
// 参数 buf：
// 参数 buflen：
// 参数 result：
// 参数 h_errnop：
int gethostbyname_r(const char *name,
                    struct hostent *ret,
                    char *buf,
                    size_t buflen,
                    struct hostent **result,
                    int *h_errnop);
```

使用示例，摘自 <https://stackoverflow.com/questions/6517478/how-to-use-gethostbyname-r-in-linux>，

```c
int rc, err;
char *str_host;
struct hostent hbuf;
struct hostent *result;

while ((rc = gethostbyname_r(str_host, &hbuf, buf, len, &result, &err)) == ERANGE) {
    /* expand buf */
    len *= 2;
    void *tmp = realloc(buf, buflen);
    if (NULL == tmp) {
        free(buf);
        perror("realloc");
    }else{
        buf = tmp;
    }
}

if (0 != rc || NULL == result) {
    perror("gethostbyname");
}
```

另一个例子，

```c++
struct sockaddr_in server;

int rc, err;
struct hostent host;
struct hostent *result;
char buf[10240];
rc = gethostbyname_r(m_host.c_str(), &host, buf, sizeof(buf), &result, &err);
if (rc != 0 || result == 0)
{
    return false;
}

memset(&server, 0, sizeof(server));
server.sin_family = AF_INET;
server.sin_port = htons(m_port);
server.sin_addr = *((struct in_addr *)host.h_addr);
int error = connect(m_socketfd, (struct sockaddr *)&server, sizeof(struct sockaddr));
if (error == -1)
{
    return false;
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

// 返回：0 为成功，其它都为失败（可以使用 gai_strerror(int) 把错误码转换为易读的字符串格式）
// 参数 node：
// 参数 service：
// 参数 hints：可以为空。
// 参数 res：结果集
int getaddrinfo(const char *node,
                const char *service,
                const struct addrinfo *hints,
                struct addrinfo **res);
                
struct addrinfo
{
    int              ai_flags;
    int              ai_family;
    int              ai_socktype;
    int              ai_protocol;
    socklen_t        ai_addrlen;
    struct sockaddr *ai_addr;
    char            *ai_canonname;
    struct addrinfo *ai_next;
};
```

- node：域名（www.example.com）、主机名、点分十进制的 ip。 如果 hints.ai_flags 中设置了 AI_NUMERICHOST 标志，那么该参数只能是点分十进制，不能是域名，该标志的作用就是阻止进行域名解析。nodename 和 servname 可以设置为 NULL，但是不能同时为 NULL。
- service：端口号、服务名（比如 "http"、"echo"...）。如果 hints.ai_flags 设置了 AI_NUMERICSERV 标志并且该参数未设置为 NULL，那么该参数必须是一个指向 10 进制的端口号字符串，不能设定成服务名，该标志就是用来阻止服务名解析。
- hints：该参数指向用户设定的 struct addrinfo 结构体，只能设定该结构体中 ai_family、ai_socktype、ai_protocol 和 ai_flags 四个域，其他域必须设置为 0 或者 NULL, 通常是申请结构体变量后使用memset 初始化再设定指定的四个域。该参数可以设置为 NULL，等价于 ai_socktype = 0， ai_protocol = 0，ai_family = AF_UNSPEC， ai_flags = 0 （在 GNU Linux 中ai_flag = AI_V4MAPPED | AI_ADDRCONFIG,可以看作是 GNU 的改进）
- res：该参数获取一个指向存储结果的 struct addrinfo 结构体列表，使用完成后调用 freeaddrinfo 释放存储结果空间。

对于 addrinfo 各成员的分析，

- ai_family，指定返回地址的协议簇，取值范围：AF_INET(IPv4)、AF_INET6(IPv6)、AF_UNSPEC(IPv4 and IPv6)
- ai_socktype，具体类型请查看 struct addrinfo 中的 `enum __socket_type` 类型，用于设定返回地址的 socke t类型，常用的有 SOCK_STREAM、SOCK_DGRAM、SOCK_RAW, 设置为 0 表示所有类型都可以。
- ai_protocol，具体取值范围请查看 Ip Protocol ，常用的有 IPPROTO_TCP、IPPROTO_UDP 等，设置为 0 表示所有协议。
- ai_flags，附加选项，多个选项可以使用或操作进行结合，具体取值范围请查看 struct addrinfo ，下面详细介绍它的选项。

**AI_PASSIVE**，如果设置了 AI_PASSIVE 标志，并且 node 是 NULL, 那么返回的 socket 地址可以用于 bind 函数，返回的地址是通配符地址(wildcard address, IPv4 时是 INADDR_ANY，IPv6 时是IN6ADDR_ANY_INIT)，这样应用程序(典型是 server)就可以使用这个通配符地址用来接收任何请求主机地址的连接；如果 node 不是 NULL，那么 AI_PASSIVE 标志被忽略。

如果未设置 AI_PASSIVE 标志，返回的 socket 地址可以用于 connect(), sendto(), 或者 sendmsg() 函数。如果 node 是 NULL，那么网络地址会被设置为 lookback 接口地址(IPv4 时是 INADDR_LOOPBACK，IPv6 时是 IN6ADDR_LOOPBACK_INIT)，这种情况下，应用是想与运行在同一个主机上另一个应用通信。

**AI_CANONNAME**，请求 canonical(主机的 official name)名字。如果设置了该标志，那么 res 返回的第一个 struct addrinfo 中的 ai_canonname 域会存储 official name 的指针。

**AI_NUMERICHOST**，阻止域名解析，具体见参数 node 中的说明。

**AI_NUMERICSERV**，阻止服务名解析，具体见参数 service 中的说明。

**AI_V4MAPPED**，当 ai_family 指定为 AF_INT6(IPv6) 时，如果没有找到 IPv6 地址，那么会返回 IPv4-mapped IPv6 地址。也就是说如果没有找到 AAAA record(用来将域名解析到 IPv6 地址的 DNS 记录),那么就查询 A record(IPv4)，将找到的 IPv4 地址映射到 IPv6 地址, IPv4-mapped IPv6 地址其实是 IPv6 内嵌 IPv4 的一种方式，地址的形式为 "0::FFFF:a.b.c.d"，例如 "::ffff:192.168.89.9"(混合格式)这个地址仍然是一个 IPv6 地址，只是"0000:0000:0000:0000:0000:ffff:c0a8:5909"(16 机制格式)的另外一种写法罢了。

当 ai_family 不 是AF_INT6(IPv6)时，该标志被忽略。

**AI_ALL**，查询 IPv4 和 IPv6 地址。

**AI_ADDRCONFIG**，只有当主机配置了 IPv4 地址才进行查询 IPv4 地址；只有当主机配置了 IPv6 地址才进行查询 IPv6 地址。

例子，

```c++
struct addrinfo *result = NULL, hints;

memset(&hints, 0 , sizeof(hints));
hints.ai_family = AF_INET;
hints.ai_socktype = SOCK_STREAM;
hints.ai_protocol = IPPROTO_TCP;
int ret = getaddrinfo("www.baidu.com", 0, &hints, &result);
if (ret)
{
    return false;
}

char ip[64] = {0};
if (inet_ntop(AF_INET, &(((struct sockaddr_in *)(result->ai_addr))->sin_addr), ip, 64))
{
    m_host = ip; // 转成点分十进制
    freeaddrinfo(result);
    return true;
}

freeaddrinfo(result);
```

另一个例子，

```c++
struct addrinfo *result = NULL, hints;

memset(&hints, 0 , sizeof(hints));
hints.ai_family = AF_INET;
hints.ai_socktype = SOCK_STREAM;
hints.ai_protocol = IPPROTO_TCP;
int ret = getaddrinfo(m_host.c_str(), Int_To_String(m_port).c_str(), &hints, &result);
if (ret)
{
    return false;
}

ret = connect(m_socketfd, result->ai_addr, result->ai_addrlen);
if (ret == -1)
{
    freeaddrinfo(result);
    return false;
}

freeaddrinfo(result);
```


参考：

- <https://man7.org/linux/man-pages/man3/getaddrinfo.3.html>
- <https://en.wikipedia.org/wiki/Getaddrinfo>
- <https://gist.github.com/jirihnidek/bf7a2363e480491da72301b228b35d5d>
- <https://www.cnblogs.com/welhzh/p/12123373.html>
- <https://blog.csdn.net/prettyshuang/article/details/50457086>

注：Windows 端的使用可以参考 MSDN 上的完整例子 [server](https://docs.microsoft.com/zh-cn/windows/win32/winsock/complete-server-code?redirectedfrom=MSDN)
 和 [client](https://docs.microsoft.com/zh-cn/windows/win32/winsock/complete-client-code?redirectedfrom=MSDN)。
