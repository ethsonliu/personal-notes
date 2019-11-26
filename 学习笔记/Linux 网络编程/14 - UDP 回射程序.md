特点：

1. 无连接，不需三次握手，，，
2. 基于消息的数据传输服务，没有黏包问题
3. 不可靠，数据包重复，丢失，乱序，，，
4. 一般情况下 udp 更高效

注意点：

1. udp 报文可能会丢失、重复
2. udp 报文可能会乱序；
3. udp 缺乏流量控制，如果缓冲区满了就直接覆盖；
4. udp 协议数据报文截断，如果发送的数据大于接收缓冲区，则多余的部分会被丢弃；
5. recvfrom 返回 0，不代表连接关闭，因为 udp 是无连接的；
6. ICMP 异步错误；
7. 

代码如下：

```c
// server
#include <iostream>
#include <unistd.h>
#include <stdlib.h>
#include <error.h>
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>
#include <cstring>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define ERR_EXIT(m) \
        do \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while (0);

void test(int sock, struct sockaddr_in* peeraddr, char* recvbuf, int n)
{
    //sendto(sock, "server", sizeof("server"), 0, (struct sockaddr *)&peeraddr, sizeof peeraddr);
    sendto(sock, recvbuf, n, 0, (struct sockaddr *)&peeraddr, sizeof peeraddr);
}
void echo_srv(int sock)
{
    char recvbuf[1024] = {0};
    struct sockaddr_in peeraddr;
    socklen_t peerlen;
    int n;
    while (1)
    {
        peerlen = sizeof peeraddr;
        memset(recvbuf, 0, sizeof recvbuf);
        n = recvfrom(sock, recvbuf, sizeof(recvbuf), 0, (struct sockaddr *)&peeraddr, &peerlen);
        printf("id = %s, ", inet_ntoa(peeraddr.sin_addr));
        printf("port = %d\n", ntohs(peeraddr.sin_port));
        if (n == -1)
        {
            if (errno == EINTR)
            {
                continue;
            }
            ERR_EXIT("recvfrom")
        }
        else if (n > 0)
        {
            fputs(recvbuf, stdout);
            //sendto(sock, recvbuf, n, 0, (struct sockaddr *)&peeraddr, peerlen);
            //sendto(sock, "h", sizeof("h"), 0, (struct sockaddr *)&peeraddr, sizeof peeraddr);
            test(sock, &peeraddr, recvbuf, n);
        }
    }
    close(sock);
};

int main(void)
{
    int sock;
    if ((sock = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        ERR_EXIT("socket");
    }
    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof servaddr);
    servaddr.sin_family = PF_INET;
    servaddr.sin_port = htons(6666);
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

    if (bind(sock, (struct sockaddr *)&servaddr, sizeof servaddr) < 0)
    {
        ERR_EXIT("bind");
    }

    echo_srv(sock);

    return 0;
}
```

```c
// client
#include <iostream>
#include <unistd.h>
#include <stdlib.h>
#include <error.h>
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>
#include <cstring>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>

using namespace std;

#define ERR_EXIT(m) \
        do \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while (0);

int main(void)
{
    int sock;
    if ((sock = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        ERR_EXIT("socket");
    }

    struct sockaddr_in server_addr;
    int srvlen;
    memset(&server_addr, 0, sizeof server_addr);
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(6666);
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    struct sockaddr_in localaddr;
    socklen_t addrlen = sizeof localaddr;
    if (getsockname(sock, (struct sockaddr*)&localaddr, &addrlen) < 0)
    {
        ERR_EXIT("getsockname");
    }
    printf("id = %s, ", inet_ntoa(localaddr.sin_addr));
    printf("port = %d\n", ntohs(localaddr.sin_port));

    char sendbuf[1024];
    int ret;
    char recvbuf[1024];
    while (fgets(sendbuf, sizeof sendbuf, stdin) != NULL)
    {
        srvlen = sizeof server_addr;
        sendto(sock, sendbuf, sizeof(sendbuf), 0, (struct sockaddr *)&server_addr, srvlen);
        // cout << sendbuf << endl;
        ret = recvfrom(sock, recvbuf, sizeof(recvbuf), 0, NULL, NULL);
        if (ret == -1)
        {
            if (errno == EINTR)
            {
                continue;
            }
            ERR_EXIT("recvfrom")
        }
        else if (ret > 0)
        {
            fputs(recvbuf, stdout);
            // sendto(sock, recvbuf, n, 0, (struct sockaddr *)&peeraddr, peerlen);
        }
        memset(sendbuf, 0, sizeof sendbuf);
        memset(recvbuf, 0, sizeof recvbuf);
    }

    return 0;
}
```
