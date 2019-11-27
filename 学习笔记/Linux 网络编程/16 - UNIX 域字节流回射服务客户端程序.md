## 目录

- [UNIX 域协议特点](#UNIX-域协议特点)
- [UNIX 域地址结构](#UNIX-域地址结构)
- [UNIX 域字节流回射服务客户端程序](# UNIX-域字节流回射服务客户端程序)
- [UNIX 域套接字编程注意点](#UNIX-域套接字编程注意点)

## UNIX 域协议特点

- UNIX 域套接字与 TCP 相比, 在同一台主机上, UNIX 域套接字更有效率, 几乎是 TCP 的两倍(由于 UNIX 域套接字不需要经过网络协议栈,不需要打包/拆包,计算
校验和,维护序号和应答等,只是将应用层数据从一个进程拷贝到另一个进程, 而且 UNIX 域协议机制本质上就是可靠的通讯,  而网络协议是为不可靠的通讯设计的)。
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



## UNIX 域套接字编程注意点












