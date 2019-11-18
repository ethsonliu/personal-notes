## 目录

- [TCP 11 种状态](#TCP-11-种状态)
- [TIME_WAIT 和 SO_REUSEADDR 状态](#TIME_WAIT-和-SO_REUSEADDR-状态)
- [信号 SIGPIPE](#信号-SIGPIPE)

## TCP 11 种状态

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/017.png)

- CLOSED：表示关闭状态（初始状态）。
- LISTEN：该状态表示服务器端的某个 SOCKET 处于监听状态，可以接受连接。
- SYN_SENT：这个状态与 SYN_RCVD 遥相呼应，当客户端 SOCKET 执行 CONNECT 连接时，它首先发送 SYN 报文，随即进入到了 SYN_SENT 状态，并等待服务端的发送三次握手中的第 2 个报文。SYN_SENT 状态表示客户端已发送 SYN 报文。
- SYN_RCVD: 该状态表示接收到S YN 报文，在正常情况下，这个状态是服务器端的 SOCKET 在建立 TCP 连接时的三次握手会话过程中的一个中间状态，很短暂。此种状态时，当收到客户端的 ACK 报文后，会进入到 ESTABLISHED 状态。
- ESTABLISHED：表示连接已经建立。
- FIN_WAIT_1: FIN_WAIT_1 和 FIN_WAIT_2 状态的真正含义都是表示等待对方的 FIN 报文。区别是： FIN_WAIT_1 状态是当 socket 在 ESTABLISHED 状态时，想主动关闭连接，向对方发送了 FIN 报文，此时该 socket 进入到 FIN_WAIT_1 状态。 FIN_WAIT_2 状态是当对方回应 ACK 后，该 socket 进入到 FIN_WAIT_2 状态，正常情况下，对方应马上回应 ACK 报文，所以 FIN_WAIT_1 状态一般较难见到，而 FIN_WAIT_2 状态可用 netstat 看到。
- FIN_WAIT_2：主动关闭链接的一方，发出 FIN 收到 ACK 以后进入该状态。称之为半连接或半关闭状态。该状态下的 socket 只能接收数据，不能发。
- TIME_WAIT: 表示收到了对方的 FIN 报文，并发送出了 ACK 报文，等 2MSL 后即可回到 CLOSED 可用状态。如果 FIN_WAIT_1 状态下，收到对方同时带 FIN 标志和 ACK 标志的报文时，可以直接进入到 TIME_WAIT 状态，而无须经过 FIN_WAIT_2 状态。
- CLOSE_WAIT: 此种状态表示在等待关闭。当对方关闭一个 SOCKET 后发送 FIN 报文给自己，系统会回应一个 ACK 报文给对方，此时则进入到 CLOSE_WAIT 状态。接下来呢，察看是否还有数据发送给对方，如果没有可以 close 这个 SOCKET，发送 FIN 报文给对方，即关闭连接。所以在 CLOSE_WAIT 状态下，需要关闭连接。
- LAST_ACK: 该状态是被动关闭一方在发送 FIN 报文后，最后等待对方的 ACK 报文。当收到 ACK 报文后，即可以进入到 CLOSED 可用状态。
- CLOSIN: 这种状态较特殊，属于一种较罕见的状态。正常情况下，当你发送 FIN 报文后，按理来说是应该先收到（或同时收到）对方的 ACK 报文，再收到对方的 FIN 报文。但是 CLOSING 状态表示你发送 FIN 报文后，并没有收到对方的 ACK 报文，反而却也收到了对方的 FIN 报文。什么情况下会出现此种情况呢？如果双方几乎在同时 close 一个 SOCKET 的话，那么就出现了双方同时发送 FIN 报文的情况，也即会出现 CLOSING 状态，表示双方都正在关闭 SOCKET 连接。

## TIME_WAIT 和 SO_REUSEADDR 状态

## 信号 SIGPIPE
