```c++
// Pub.h
#ifndef NETWORKPROGRAMMING_PUB_H
#define NETWORKPROGRAMMING_PUB_H

#include <list>
#include <algorithm>

using namespace std;

// C2S
#define C2S_LOGIN               0x01
#define C2S_LOGOUT              0x02
#define C2S_ONLINE_USER         0x03

#define MSG_LEN                 512

// S2C
#define S2C_LOGIN_OK            0x01
#define S2C_ALREADY_LOGINED     0x02
#define S2C_SOMEONE_LOGIN       0x03
#define S2C_SOMEONE_LOGOUT      0x04
#define S2C_ONLINE_USER         0x05

// C2C
#define C2C_CHAT                0x06

typedef struct message
{
    int cmd;
    char body[MSG_LEN];
} MESSAGE;

typedef struct user_info
{
    char username[16];
    unsigned int ip;
    unsigned short port;
} USER_INFO;

typedef struct char_msg
{
    char username[16];
    char msg[100];
} CHAT_MSG;

typedef list<USER_INFO> USER_LIST;

#endif //NETWORKPROGRAMMING_PUB_H
```

```c++
// server
#include "Pub.h"
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

// 聊天室成员列表
USER_LIST client_list;

void test(int sock, struct sockaddr_in* peeraddr)
{

    sendto(sock, "server", sizeof("server"), 0, (struct sockaddr *)&peeraddr, sizeof peeraddr);
}

void do_login(MESSAGE msg, int sock, struct sockaddr_in cliaddr)
{
    // cout << "do_login" << endl;
    USER_INFO user;
    strcpy(user.username, msg.body);
    user.ip = cliaddr.sin_addr.s_addr;
    user.port = cliaddr.sin_port;

    /* 查找用户 */
    USER_LIST::iterator it;
    for (it = client_list.begin(); it != client_list.end(); ++it)
    {
        if (strcmp(it->username, msg.body) == 0)
        {
            break;
        }
    }

    // 未找到用户
    if (it == client_list.end())
    {
        printf("has a user login: %s <-> %s:%d\n", msg.body, inet_ntoa(cliaddr.sin_addr), cliaddr.sin_port);
        client_list.push_back(user);

        // 登录成功应答
        MESSAGE reply_msg;
        memset(&reply_msg, 0, sizeof(reply_msg));
        reply_msg.cmd = htons(S2C_LOGIN_OK);
        sendto(sock, &msg, sizeof(msg), 0, (struct sockaddr *)&cliaddr, sizeof cliaddr);

        int count = htons((int)client_list.size());
        // 发送在线人数
        sendto(sock, &count, sizeof(int), 0, (struct sockaddr*)&cliaddr, sizeof cliaddr);

        printf("sending user list information to: %s <-> %s:%d\n", msg.body, inet_ntoa(cliaddr.sin_addr), cliaddr.sin_port);
        // 发送在线列表
        for (it = client_list.begin(); it != client_list.end(); ++it)
        {
            sendto(sock, &*it, sizeof(USER_INFO), 0, (struct sockaddr*)&cliaddr, sizeof cliaddr);
        }

        // 向其他用户通知有新用户登录
        for (it = client_list.begin(); it != client_list.end(); ++it)
        {
            if (strcmp(it->username, msg.body) == 0)    // 如果是当前用户，则跳过
            {
                continue;
            }

            struct sockaddr_in peeraddr;
            memset(&peeraddr, 0, sizeof peeraddr);
            peeraddr.sin_family = AF_INET;
            peeraddr.sin_port = it->port;
            peeraddr.sin_addr.s_addr = it->ip;

            msg.cmd = htons(S2C_SOMEONE_LOGIN);
            memcpy(msg.body, &user, sizeof(user));

            if (sendto(sock, &msg, sizeof(msg), 0, (struct sockaddr*)&peeraddr, sizeof peeraddr) < 0)
            {
                ERR_EXIT("sendto");
            }
        }

    }
    else    // 找到该用户  fixme[如果找到该用户之后还有其他操作吗?]
    {
        printf("user %s has already logined\n", msg.body);

        MESSAGE reply_msg;
        memset(&reply_msg, 0, sizeof(MESSAGE));
        reply_msg.cmd = htons(S2C_ALREADY_LOGINED);
       // int a = ntohs(reply_msg.cmd);
        sendto(sock, &reply_msg, sizeof(MESSAGE), 0, (struct sockaddr*)&cliaddr, sizeof cliaddr);
    }
}

void do_logout(MESSAGE msg, int sock, struct sockaddr_in cliaddr)
{
     //cout << "do_logout" << endl;
     printf("has a user logout : %s <-> %s:%d\n", msg.body, inet_ntoa(cliaddr.sin_addr), cliaddr.sin_port);

     USER_LIST::iterator it;
     for (it = client_list.begin(); it != client_list.end(); ++it)
     {
        if (strcmp(it->username, msg.body) == 0)
        {
            break;
        }
     }

     if (it != client_list.end())
     {
         client_list.erase(it);
     }

     // 向其他用户通知有用户退出
     for (it = client_list.begin(); it != client_list.end(); ++it)
     {
         if (strcmp(it->username, msg.body) == 0)
         {
             continue;
         }
         struct sockaddr_in peeraddr;
         memset(&peeraddr, 0, sizeof peeraddr);
         peeraddr.sin_family = AF_INET;
         peeraddr.sin_addr.s_addr = it->ip;
         peeraddr.sin_port = it->port;

         MESSAGE m;
         strcpy(m.body, msg.body);
         m.cmd = S2C_SOMEONE_LOGOUT;
         if (sendto(sock, &m, sizeof(MESSAGE), 0, (struct sockaddr*)&peeraddr, sizeof(peeraddr)) < 0)
         {
             ERR_EXIT("sendto");
         }
     }
}

void do_sendlist(int sock, struct sockaddr_in cliaddr)
{
    //cout << "do_sendlist" << endl;
    MESSAGE msg;
    msg.cmd = htons(S2C_ONLINE_USER);
    sendto(sock, (const char*)&msg, sizeof(msg), 0, (struct sockaddr*)&cliaddr, sizeof cliaddr);

    int count = htons((int)client_list.size());
    /* 发送在线用户数 */
    sendto(sock, (const char*)&count, sizeof (int), 0, (struct sockaddr*)&cliaddr, sizeof cliaddr);
    /* 发送在线用户列表 */
    for (USER_LIST::iterator it = client_list.begin(); it != client_list.end(); ++it)
    {
        sendto(sock, &*it, sizeof(USER_INFO), 0, (struct sockaddr*)&cliaddr, sizeof cliaddr);
    }
}

void chat_srv(int sock)
{
    struct sockaddr_in cliaddr;
    socklen_t clilen;
    int n;
    MESSAGE msg;
    while (1)
    {
        memset(&msg, 0, sizeof(msg));
        clilen = sizeof cliaddr;
        n = recvfrom(sock, &msg, sizeof(msg), 0, (struct sockaddr *)&cliaddr, &clilen);
        if (n == -1)
        {
            if (errno == EINTR)
            {
                continue;
            }
            ERR_EXIT("recvfrom")
        }

        int cmd = ntohs(msg.cmd);
        switch (cmd)
        {
            case C2S_LOGIN:
                do_login(msg, sock, cliaddr);
                break;
            case C2S_LOGOUT:
                do_logout(msg, sock, cliaddr);
                break;
            case C2S_ONLINE_USER:
                do_sendlist(sock, cliaddr);
                break;
            default:
                break;
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

    chat_srv(sock);

    return 0;
}
```

```c++
// client
#include "Pub.h"
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

char username[16];
USER_LIST client_list;

#define ERR_EXIT(m) \
        do \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while (0);


void do_someone_login(MESSAGE msg)
{
    // puts("do_someone_login\n");
    USER_INFO *user = (USER_INFO*)msg.body;
    in_addr tmp;
    tmp.s_addr = user->ip;
    printf("%s <-> %s : %d has logined server\n", user->username, inet_ntoa(tmp), ntohs(user->port));
    client_list.push_back(*user);
}

void do_someone_logout(MESSAGE msg)
{
    // puts("do_someone_logout\n");
    USER_LIST::iterator it;
    for (it = client_list.begin(); it != client_list.end(); ++it)
    {
        if (strcmp(it->username, msg.body) == 0)
        {
            break;
        }
    }

    if (it != client_list.end())
    {
        client_list.erase(it);
    }
    printf("user %s has logout\n", msg.body);
}

void do_getlist(int sock)
{
    // puts("do_getlist\n");
    int count;
    recvfrom(sock, &count, sizeof(int), 0, NULL, NULL);
    printf("has %d user logined server\n", ntohs(count));
    client_list.clear();

    int n = ntohs(count);
    for (int i = 0; i < n; ++i)
    {
        USER_INFO user;
        recvfrom(sock, &user, sizeof (USER_INFO), 0, NULL, NULL);
        client_list.push_back(user);
        in_addr tmp;
        tmp.s_addr = user.ip;

        printf("%s <-> %s : %d\n", user.username, inet_ntoa(tmp), ntohs(user.port));
    }
}

char* trim(char* cmdline)
{
    char* s = cmdline;
    while (*s == ' ')
    {
        s++;
    }

    char* e = s;
    int len = 0;
    while ((strcmp(e, "") != 0) && *e != ' ')
    {
        len++;
        e++;
    }
    char* res = new char[len];
    memcpy(res, s, len);
    return res;
}

vector<char*> getcmd(char* cmdline) {
    vector<char*> res;
    char *s = trim(cmdline);
    if (strcmp(s, "") == 0) {
        return res;
    }
    int len = 0;
    char *temp;
    char *tmp ;
    int count = 3;
    while (count--) {
        s = trim(cmdline);
        if (strcmp(s, "") == 0) {
            return res;
        }
        len = 0;
        temp = s;
        tmp = new char[500];
        while ((strcmp(temp, "") != 0) && *temp != ' ') {
            len++;
            temp++;
        }
        memcpy(tmp, s, len);
        res.push_back(tmp);
        cmdline = cmdline + len;
        while ((strcmp(cmdline, "") != 0) && *cmdline == ' ')
        {
            cmdline++;
        }
    }
    return res;
}

bool sendmsgto(int sock, char* name, char* msg)
{
    //cout << "sendmsgto" << endl;
//    if (strcmp(name, username) == 0)
//    {
//        printf("can't send message to self\n");
//        return false;
//    }

    USER_LIST::iterator it;
    for (it = client_list.begin(); it != client_list.end(); ++it)
    {
        if (strcmp(it->username, name) == 0)
        {
            break;
        }
    }

    if (it == client_list.end())
    {
        printf("user %s has not logined server\n", name);
        return false;
    }

    MESSAGE m;
    memset(&m, 0, sizeof(m));
    m.cmd = htons(C2C_CHAT);

    CHAT_MSG cm;
    strcpy(cm.username, username);
    strcpy(cm.msg, msg);

    memcpy(m.body, &cm, sizeof(MESSAGE));

    struct sockaddr_in peeraddr;
    memset(&peeraddr, 0, sizeof peeraddr);
    peeraddr.sin_family = AF_INET;
    peeraddr.sin_addr.s_addr = it->ip;
    peeraddr.sin_port = it->port;

    in_addr tmp;
    tmp.s_addr = it->ip;

    printf("sending message [%s] to user [%s] <-> %s:%d\n", msg, it->username, inet_ntoa(peeraddr.sin_addr), it->port);
    sendto(sock, &m, sizeof(m), 0, (struct sockaddr*)&peeraddr, sizeof peeraddr);   // 直接发送给客户
    return true;
}

void parse_cmd(char *cmdline, int sock, struct sockaddr_in server_addr)
{
    // cout << "parse_cmd" << endl;
    //char cmd[10] = {0};
    //char* cmd;
//    char* p;
//    p = strchr(cmdline, ' ');   // 得到' '字符出现的第一个位置
//    if (p != NULL)
//    {
//        *p = '\0';
//    }
//    strcmp(cmd, cmdline);
    //cmd = trim(cmdline);
    vector<char*> cmd = getcmd(cmdline);    // [0]为命令 [1]为发送对方 [2]为发送消息
    if (cmd.size() == 0 || cmd.size() == 2)
    {
        printf("bad command\n");
        printf("\n Commands are:\n");
        printf("send username msg\n");
        printf("list\n");
        printf("exit\n");
        printf("\n");
    }
    else if (strcmp(cmd[0], "exit") == 0)
    {
        MESSAGE msg;
        memset(&msg, 0, sizeof(msg));
        msg.cmd = htons(C2S_LOGOUT);
        strcpy(msg.body, username);

        if (sendto(sock, &msg, sizeof(MESSAGE), 0, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0)
        {
            ERR_EXIT("sendto");
        }
        printf("user %s has logout server\n", username);
        exit(EXIT_SUCCESS);
    }
    else if (strcmp(cmd[0], "send") == 0)
    {
        char* peername = new char[16];
        char* msg = new char[MSG_LEN];

        peername = cmd[1];
        msg = cmd[2];
        /* send user msg */
        /*      p    p2  */
        sendmsgto(sock, peername, msg);
    }
    else if (strcmp(cmd[0], "list") == 0)
    {
        MESSAGE msg;
        memset(&msg, 0, sizeof(MESSAGE));
        msg.cmd = htons(C2S_ONLINE_USER);

        if (sendto(sock, &msg, sizeof(MESSAGE), 0, (struct sockaddr*)&server_addr, sizeof server_addr) < 0)
        {
            ERR_EXIT("sendto");
        }
    }
    else
    {
        printf("bad command\n");
        printf("\n Commands are:\n");
        printf("send username msg\n");
        printf("list\n");
        printf("exit\n");
        printf("\n");
    }
};

void do_chat(const MESSAGE& msg)
{
    // cout << "do_chat" << endl;
    CHAT_MSG *cm = (CHAT_MSG*)msg.body;
    printf("recv a msg [%s] from [%s]\n", cm->msg, cm->username);
}

void chat_cli(int sock)
{
    struct sockaddr_in server_addr;

    memset(&server_addr, 0, sizeof server_addr);
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(6666);
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    MESSAGE msg;
    int ret;
    while (1)
    {
        memset(&msg, 0, sizeof(msg));
        cout << "please input your name:";
        fflush(stdout);
        scanf("%s", username);

        memset(&msg, 0, sizeof(msg));
        msg.cmd = ntohs(C2S_LOGIN);
        strcpy(msg.body, username);

        sendto(sock, &msg, sizeof(msg), 0, (struct sockaddr *)&server_addr, sizeof server_addr);

        memset(&msg, 0, sizeof(msg));
        ret = recvfrom(sock, &msg, sizeof(msg), 0, NULL, NULL);
        if (ret == -1)
        {
            if (errno == EINTR)
            {
                continue;
            }
            ERR_EXIT("recvfrom")
        }

        int cmd = ntohs(msg.cmd);
        if (cmd == S2C_ALREADY_LOGINED)
        {
            printf("user %s already logined server, please use another user\n", msg.body);
        }
        else if (cmd == S2C_LOGIN_OK)
        {
            printf("user %s has logined server\n", username);
            break;
        }
    }

    int count = 0;
    recvfrom(sock, &count, sizeof(int), 0, NULL, NULL);

    int n = ntohs(count);
    printf("has %d users logined server\n", n);

    for (int i = 0; i < n; ++i)
    {
        USER_INFO user;
        recvfrom(sock, &user, sizeof(USER_INFO), 0, NULL, NULL);
        client_list.push_back(user);
        in_addr tmp;
        tmp.s_addr = user.ip;

        printf("%d %s <-> %s:%d\n", i, user.username, inet_ntoa(tmp), user.port);
    }
    printf("\n Commands are:\n");
    printf("send username msg\n");
    printf("list\n");
    printf("exit\n");
    printf("\n");

    fd_set rset;
    FD_ZERO(&rset);
    int nready;
    socklen_t peerlen;
    struct sockaddr_in peeraddr;
    while (1)
    {
        FD_SET(STDIN_FILENO, &rset);
        FD_SET(sock, &rset);
        nready = select(sock+1, &rset, NULL, NULL, NULL);
        if (nready == -1)
        {
            ERR_EXIT("select");
        }
        if (nready == 0)
        {
            continue;
        }
        if (FD_ISSET(sock, &rset))
        {
            peerlen = sizeof(peeraddr);
            memset(&msg, 0, sizeof(msg));
            recvfrom(sock, &msg, sizeof(msg), 0, (struct sockaddr *)&peeraddr, &peerlen);
            int cmd = ntohs(msg.cmd);
            switch (cmd)
            {
                case S2C_SOMEONE_LOGIN:
                    do_someone_login(msg);
                    break;
                case S2C_SOMEONE_LOGOUT:
                    do_someone_logout(msg);
                    break;
                case S2C_ONLINE_USER:
                    do_getlist(sock);
                    break;
                case C2C_CHAT:
                    do_chat(msg);
                    break;
                default:
                    break;
            }
        }
        if (FD_ISSET(STDIN_FILENO, &rset))
        {
            char cmdline[100] = {0};
            if (fgets(cmdline, sizeof(cmdline), stdin) == NULL)
            {
                break;
            }
            if (cmdline[0] == '\n')
            {
                continue;
            }
            cmdline[strlen(cmdline) - 1] = '\0';
            parse_cmd(cmdline, sock, server_addr);
        };
    }
}

int main(void)
{
    int sock;
    if ((sock = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        ERR_EXIT("socket");
    }
    chat_cli(sock);

    return 0;
}
```
