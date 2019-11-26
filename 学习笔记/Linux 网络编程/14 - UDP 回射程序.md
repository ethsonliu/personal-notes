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
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/select.h>
#include <string.h>
#include <strings.h>

#define SERV_PORT 8888
#define MAXLINE 1024

int main()
{
    int sockfd;
    struct sockaddr_in servaddr, cliaddr;
    sockfd = socket(AF_INET, SOCK_DGRAM, 0); //创建套接字
    bzero(&servaddr, sizeof(servaddr)); //初始化地址结构
    
    //向servaddr结构写入地址
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERV_PORT);
    
    //将socket与地址绑定
    bind(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr));

    int n;
    char mesg[MAXLINE];
    socklen_t len;
    
    //消息处理循环
    while(1)
    {
        len = sizeof(cliaddr);
        
        //接收数据包
        n = recvfrom(sockfd, mesg, MAXLINE, 0, (struct sockaddr*)&cliaddr, &len);
        
        //将数据包发回给源地址
        sendto(sockfd, mesg, n, 0, (struct sockaddr*)&cliaddr, len);
    }
}
```

```c
// client
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/select.h>
#include <string.h>
#include <strings.h>

#define SERV_PORT 8888
#define MAXLINE 1024

int main(int argc, char **argv)
{
    int sockfd;
    struct sockaddr_in servaddr;

    sockfd = socket(AF_INET, SOCK_DGRAM, 0); //创建套接字
    
    //写入服务器地址，IP地址由程序的输入获取
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERV_PORT);
    inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

    int n;
    char recvline[MAXLINE+1], sendline[MAXLINE];
    
    //发送消息循环
    while(fgets(sendline, MAXLINE, stdin) != NULL)
    {
        sendto(sockfd, sendline, strlen(sendline), 0, (struct sockaddr*)&servaddr, sizeof(servaddr));
        n = recvfrom(sockfd, recvline, MAXLINE, 0, NULL, NULL);
        recvline[n] = '\0';
        fputs(recvline, stdout);
    }
}
```
