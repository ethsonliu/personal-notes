## 目录

- [TCP 11 种状态](#TCP-11-种状态)
- [TIME_WAIT 和 SO_REUSEADDR 状态](#TIME_WAIT-和-SO_REUSEADDR-状态)
- [信号 SIGPIPE](#信号-SIGPIPE)

## TCP 11 种状态

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/017.png)

- **CLOSED**：表示关闭状态（初始状态）。
- **LISTEN**：该状态表示服务器端的某个 SOCKET 处于监听状态，可以接受连接。
- **SYN_SENT**：这个状态与 SYN_RCVD 遥相呼应，当客户端 SOCKET 执行 CONNECT 连接时，它首先发送 SYN 报文，随即进入到了 SYN_SENT 状态，并等待服务端的发送三次握手中的第 2 个报文。SYN_SENT 状态表示客户端已发送 SYN 报文。
- **SYN_RCVD**: 该状态表示接收到S YN 报文，在正常情况下，这个状态是服务器端的 SOCKET 在建立 TCP 连接时的三次握手会话过程中的一个中间状态，很短暂。此种状态时，当收到客户端的 ACK 报文后，会进入到 ESTABLISHED 状态。
- **ESTABLISHED**：表示连接已经建立。
- **FIN_WAIT_1**: FIN_WAIT_1 和 FIN_WAIT_2 状态的真正含义都是表示等待对方的 FIN 报文。区别是： FIN_WAIT_1 状态是当 socket 在 ESTABLISHED 状态时，想主动关闭连接，向对方发送了 FIN 报文，此时该 socket 进入到 FIN_WAIT_1 状态。 FIN_WAIT_2 状态是当对方回应 ACK 后，该 socket 进入到 FIN_WAIT_2 状态，正常情况下，对方应马上回应 ACK 报文，所以 FIN_WAIT_1 状态一般较难见到，而 FIN_WAIT_2 状态可用 netstat 看到。
- **FIN_WAIT_2**：主动关闭链接的一方，发出 FIN 收到 ACK 以后进入该状态。称之为半连接或半关闭状态。该状态下的 socket 只能接收数据，不能发。
- **TIME_WAIT**: 表示收到了对方的 FIN 报文，并发送出了 ACK 报文，等 2MSL 后即可回到 CLOSED 可用状态。如果 FIN_WAIT_1 状态下，收到对方同时带 FIN 标志和 ACK 标志的报文时，可以直接进入到 TIME_WAIT 状态，而无须经过 FIN_WAIT_2 状态。
- **CLOSE_WAIT**: 此种状态表示在等待关闭。当对方关闭一个 SOCKET 后发送 FIN 报文给自己，系统会回应一个 ACK 报文给对方，此时则进入到 CLOSE_WAIT 状态。接下来呢，察看是否还有数据发送给对方，如果没有可以 close 这个 SOCKET，发送 FIN 报文给对方，即关闭连接。所以在 CLOSE_WAIT 状态下，需要关闭连接。
- **LAST_ACK**: 该状态是被动关闭一方在发送 FIN 报文后，最后等待对方的 ACK 报文。当收到 ACK 报文后，即可以进入到 CLOSED 可用状态。
- **CLOSING**: 这种状态较特殊，属于一种较罕见的状态。正常情况下，当你发送 FIN 报文后，按理来说是应该先收到（或同时收到）对方的 ACK 报文，再收到对方的 FIN 报文。但是 CLOSING 状态表示你发送 FIN 报文后，并没有收到对方的 ACK 报文，反而却也收到了对方的 FIN 报文。什么情况下会出现此种情况呢？如果双方几乎在同时 close 一个 SOCKET 的话，那么就出现了双方同时发送 FIN 报文的情况，也即会出现 CLOSING 状态，表示双方都正在关闭 SOCKET 连接。

## TIME_WAIT 和 SO_REUSEADDR 状态

## 信号 SIGPIPE

当服务器 close 一个连接时，若 client 端接着发数据。根据 TCP 协议的规定，会收到一个 RST 响应，client 再往这个服务器发送数据时，系统会发出一个 SIGPIPE 信号给进程，告诉进程这个连接已经断开了，不要再写了。

我写了一个服务器程序,在 Linux 下测试，然后用 C++ 写了客户端用千万级别数量的短链接进行压力测试.  但是服务器总是莫名退出，没有 core 文件.

最后问题确定为, 对一个对端已经关闭的 socket 调用两次 write, 第二次将会生成 SIGPIPE 信号, 该信号默认结束进程.

具体的分析可以结合 TCP 的"四次握手"关闭. TCP 是全双工的信道, 可以看作两条单工信道, TCP连 接两端的两个端点各负责一条. 当对端调用 close 时, 虽然本意是关闭整个两条信道, 但本端只是收到 FIN 包. 按照 TCP 协议的语义, 表示对端只是关闭了其所负责的那一条单工信道, 仍然可以继续接收数据. 也就是说, 因为 TCP 协议的限制, 一个端点无法获知对端的 socket 是调用了 close 还是 shutdown。

对一个已经收到 FIN 包的 socket 调用 read 方法, 如果接收缓冲已空, 则返回 0, 这就是常说的表示连接关闭. 但第一次对其调用 write 方法时, 如果发送缓冲没问题, 会返回正确写入(发送). 但发送的报文会导致对端发送 RST 报文, 因为对端的 socket 已经调用了 close, 完全关闭, 既不发送, 也不接收数据. 所以, 第二次调用 write 方法(假设在收到RST之后), 会生成 SIGPIPE 信号, 导致进程退出.

为了避免进程退出, 可以捕获 SIGPIPE 信号, 或者忽略它, 给它设置 SIG_IGN 信号处理函数:

```
signal(SIGPIPE, SIG_IGN);
```

这样, 第二次调用 write 方法时, 会返回 -1, 同时 errno 置为 SIGPIPE. 程序便能知道对端已经关闭.

在 linux 下写 socket 的程序的时候，如果尝试 send 到一个 disconnected socket 上，就会让底层抛出一个 SIGPIPE 信号。这个信号的缺省处理方法是退出进程，大多数时候这都不是我们期望的。因此我们需要重载这个信号的处理方法。调用以下代码，即可安全的屏蔽 SIGPIPE：

```
signal(SIGPIPE, SIG_IGN);
```

我的程序产生这个信号的原因是: client 端通过 pipe 发送信息到 server 端后，就关闭 client 端, 这时 server 端,返回信息给 client 端时就产生 Broken pipe 信号了，服务器就会被系统结束了。

对于产生信号，我们可以在产生信号前利用方法 signal(int signum, sighandler_t handler) 设置信号的处理。如果没有调用此方法，系统就会调用默认处理方法：中止程序，显示提示信息(就是我们经常遇到的问题)。我们可以调用系统的处理方法，也可以自定义处理方法。 

项目中我调用了 signal(SIGPIPE, SIG_IGN), 这样产生 SIGPIPE 信号时就不会中止程序，直接把这个信号忽略掉。

转载自：[SIGPIPE信号详解](https://blog.csdn.net/lmh12506/article/details/8457772)
