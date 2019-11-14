# 目录

- [read&write 与 recv&send 区别](#read&write-与-recv&send-区别)


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








