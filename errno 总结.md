## 目录

- [11 - EAGAIN](#11---EAGAIN)
- [32 - EPIPE](#32---EPIPE)

## 11 - EAGAIN

Try again / Resource temporarily unavailable 再次尝试 / 资源暂不可用

这通常由非阻塞 IO 影响的，比如 udp 的 recvfrom，你给它设置了超时，当到达超时时间，但是依旧没有数据的时候，就会出现这个错误。

当遇到这个错误的时候，不用管它，你可以接着 recvfrom 就可以。

## 32 - EPIPE

Broken pipe 管道破裂。

主要是因为 socket 被关闭后，客户端还在向这个 socket 发送数据或者读取这个 socket 的数据，这种情况一般需要应用程序执行断线重连就可以解决。如果重连依旧返回该错误，一般是服务端做了连接频率限制，可以考虑在连接前加一些延迟。

该错误发生时，会触发 SIGPIPE 信号，该信号默认行为是退出所在进程，可以参考 <https://blog.csdn.net/yockie/article/details/52176171> 来禁止默认行为。
