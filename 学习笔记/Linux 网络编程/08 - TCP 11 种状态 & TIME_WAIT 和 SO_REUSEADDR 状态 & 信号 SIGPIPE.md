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

TIME_WAIT 状态是 TCP 连接中主动关闭连接的一方会进入的状态，在发出最后一个 ACK 包之后，主动关闭方进入 TIME_WAIT 状态，从而确保 ACK 包到达对端，以及等待网络中之前迷路的数据包完全消失，防止端口被复用的时候收到迷路包从而出现收包错误。

TIME_WAIT 状态会持续 2MSL（max segment lifetime）的时间，一般 1 分钟到 4 分钟。在这段时间内端口不能被重新分配使用。

TIME_WAIT 并不会占用过多的系统资源，但是可以通过修改内核参数/ etc/sysctl.conf 来限制 TIME_WAIT 数量。

为了避免以上情况，TCP/IP 协议不允许处于 TIME_WAIT 状态的连接启动一个新的可用连接，因为 TIME_WAIT 状态持续 2MSL，这就可以保证当成功建立一个新 TCP 连接的时候，来自旧连接重复分组已经在网络中消失。

TIME_WAIT 状态存在的原因：

1. 可靠地实现 TCP 全双工连接的可靠终止。TCP 协议在关闭连接的四次握手过程中，最终 ACK 是由主动关闭连接的一端（以下简称 A 端）发出的，如果这个 ACK 丢失，对方（以下简称 B 端）将重发最终的 FIN，因此 A 端就必须维护状态信息 TIME_WAIT 允许它发送最终的 ACK。如果 A 端不维持 TIME_WAIT 的状态，而是处于 CLOSED 状态，那么 A 端将响应 RST（reset）分节，B 端收到后将此分节解释成一个异常（Java 中会抛出 connection reset 的 SocketException）。因而，要实现 TCP 全双工连接的正常终止，必须处理终止过程中四个分节中，任何一个分节丢失的情况，主动关闭连接的 A 端必须维持 TIME_WAIT 的状态。
2. 保证此次连接的重复数据段从网络中消失。TCP 分节可能由于路由器异常而“迷路”，在“迷路”期间，TCP 发送端可能因确认超时而重发这个分节，“迷路”的分节在路由器恢复正常后也会被发送到最终的目的地，这个迟到的“迷路”分节到达时可能会引起问题。在关闭“前一个连接”之后，马上又建立起一个相同的 IP 和端口之间的“新连接”，这会导致“前一个连接”的迷路重复分组在“前一个连接”终止后到达，从而被“新连接”接收到了。

在写一个 unix server 程序时，经常需要在命令行重启它，绝大多数时候工作正常，但是某些时候会抛出异常 “bind: address already in use”，于是重启失败。上面这个就是地址 reuse 问题，就是由于 TIME_WAIT 状态产生的，我们有以下方案来解决这个问题：

**SO_REUSEADDR**

这个 socket 选项通知内核：如果端口忙，但 TCP 状态位于 TIME_WAIT，可以重用端口。一个 socket 由相关五元组构成，协议、本地地址、本地端口、远程地址、远程端口。SO_REUSEADDR 仅仅表示可以重用本地本地地址、本地端口，整个相关五元组还是唯一确定的。所以，重启后的服务程序有可能收到非期望数据。必须慎重使用 SO_REUSEADDR 选项。

一般来说，一个端口释放后会等待两分钟之后才能再被使用，SO_REUSEADDR 是让端口释放后立即就可以被再次使用。

SO_REUSEADDR 用于对 TCP 处于 TIME_WAIT 状态下的 socket，才可以重复绑定使用。server 程序总是应该在调用 bind() 之前设置 SO_REUSEADDR 选项。先调用close() 的一方会进入 TIME_WAIT 状态。

SO_REUSEADDR 允许启动一个监听服务器并捆绑其众所周知端口，即使以前建立的将此端口用做他们的本地端口的连接仍存在。这通常是重启监听服务器时出现，若不设置此选项，则 bind 时将出错。

**SO_LINGER**

Linux 网络编程中，socket 的选项很多。其中几个比较重要的选项就包括 SO_LINGER。

在默认情况下,当调用 close() 关闭 socket 的使用，close() 会立即返回,但是,如果 send buffer 中还有数据，系统会试着先把 send buffer 中的数据发送出去，SO_LINGER 选项则是用来修改这种默认操作的。SO_LINGER 是一个 socket 选项，可以通过 setsockopt API 进行设置，使用起来比较简单，但其实现机制比较复杂，且字面意思上比较难理解。SO_LINGER 的值用如下数据结构表示：

```
struct linger {
int l_onoff // 0 = off, nonzero = on(开关)
int l_linger // linger time(延迟时间)
}
```

其取值和处理如下：

1. 设置 l_onoff 为 0，则该选项关闭，l_linger 的值被忽略，等于内核缺省情况，close() 调用会立即返回给调用者，如果可能将会传输任何未发送的数据；
2. 设置 l_onoff 为非 0，l_linger 为 0，当调用 close() 的时候,TCP 连接会立即断开。send buffer 中未被发送的数据将被丢弃，并向对方发送一个 RST 信息。值得注意的是，由于这种方式，不是以 4 次握手方式结束 TCP 连接，所以，TCP 连接将不会进入 TIME_WAIT 状态，这样会导致新建立的可能和旧连接的数据造成混乱。这种关闭方式称为“强制”或“失效”关闭。通常会看到“Connection reset by peer”之类的错误；
3. 设置 l_onoff 为非 0，l_linger 为非 0，在这种情况下，会使得 close() 返回得到延迟。调用 close() 去关闭 socket 的时候，内核将会延迟。也就是说，如果 send buffer 中还有数据尚未发送，该进程将会被休眠直到一下任何一种情况发生：一，send buffer 中的所有数据都被发送并且得到对方 TCP 的应答消息；二，延迟时间消耗完。在延迟时间被消耗完之后，send buffer 中的所有数据都将会被丢弃。这种关闭称为“优雅的”关闭。

因此，在正常情况下，在 socket 调用 close() 之前设置 SO_LINGER 超时为 0 都不是个好的选择。但也有些情况下需要使用 SO_LINGER：

如 server 返回无效数据或者超时时，SO_LINGER 有助于避免卡在 CLOSE_WAIT 或 TIME_WAIT 的状态；如果必须启动有数千个客户端连接的 app，则可以考虑设置 SO_LINGER，从而避免数千个 socket 处于 TIME_WAIT 状态，从而减少可用端口在服务重启后，新客户端连接收到的影响；

通过上面的讨论，我们知道 TIME_WAIT 状态是友好的，并不是多余的，TCP 要保证在所有可能的情况下使得所有的数据都能够正确送达。当你关闭一个 socket 时，主动关闭一端的 socket 将进入 TIME_WAIT 状态，而被动关闭的一方则进入 CLOSED 状态，这的确能够保证所有的数据都被传送。

当一个 socket 关闭的时候，是通过两端四次挥手完成的，当一端调用 close() 时，就说明本端没有数据要传送了，这好像看来在挥手完成以后，socket 就可以处于 CLOSED 状态了，其实不然，原因是这样安排状态有两个问题，首先我们没有任何机制保证最后的一个 ACK 能够正常传输，第二，网络仍然可能有残余的数据包，我们也必须能够正常处理。TIME_WAIT 状态就是为了解决这两个问题而生的。

服务端为了解决这个 TIME_WAIT 问题，可选的方式有3种：

1. 保证由客户端主动发起关闭
2. 关闭的时候使用 RST 方式（set SO_LINGER）
3. 对处于 TIME_WAIT 状态的 TCP 允许重用（set SO_REUSEADDR）


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
