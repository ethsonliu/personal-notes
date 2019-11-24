## 目录

- [epoll 介绍](#epoll-介绍)
- [示例代码](#示例代码)
- [select & poll & epoll 比较](#select-&-poll-&-epoll-比较)

## epoll 介绍

epoll 是Linux 下多路复用 IO 接口 select/poll 的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率，因为它会复用文件描述符集用来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集（内部红黑树实现），只要遍历那些被内核 IO 事件异步唤醒而加入 Ready 队列的描述符集合就行了。

epoll 除了提供 select/poll 那种 IO 事件的水平触发 （Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存 IO 状态，减少 epoll_wait/epoll_pwait 的调用，提高应用程序效率。

epoll 操作过程需要三个接口，分别如下：

```c
#include <sys/epoll.h>

int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

### int epoll_create(int size)

man 参考：http://man7.org/linux/man-pages/man2/epoll_create.2.html

epoll_create 建立一个 epoll 对象。参数 size 是内核保证能够正确处理的最大句柄数，多于这个最大数时内核可不保证效果。但这个参数从 Linux 2.6.8 起就被忽略了，只需大于 0 即可。

当创建好 epoll 句柄后，它就是会占用一个 fd 值，在 linux 下如果查看 /proc/id/fd/，是能够看到这个 fd 的，所以在使用完 epoll 后，必须调用 close() 关闭，否则可能导致 fd 被耗尽。

### int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)

man 参考：http://man7.org/linux/man-pages/man2/epoll_ctl.2.html

epoll 的事件注册函数，它不同于 select() 是在监听事件时告诉内核要监听什么类型的事件， epoll 是在这里先注册要监听的事件类型。

第一个参数是 epoll_create() 的返回值，第二个参数表示动作，用三个宏来表示：

```
EPOLL_CTL_ADD // 注册新的 fd 到 epfd 中
EPOLL_CTL_MOD // 修改已经注册的 fd 的监听事件
EPOLL_CTL_DEL // 从 epfd 中删除一个 fd
```

第三个参数是需要监听的 fd。

第四个参数是告诉内核需要监听什么事，struct epoll_event 结构如下：

```c
typedef union epoll_data {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
}epoll_data_t;

struct epoll_event {
    uint32_t events;   /* Epoll events */
    epoll_data_t data; /* User data variable */
};
```

events 可以是以下几个宏的集合：

```
EPOLLIN      // 表示对应的文件描述符可以读（包括对端 SOCKET 正常关闭）
EPOLLOUT     // 表示对应的文件描述符可以写
EPOLLPRI     // 表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来)
EPOLLERR     // 表示对应的文件描述符发生错误
EPOLLHUP     // 表示对应的文件描述符被挂断
EPOLLET      // 将 EPOLL 设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的
EPOLLONESHOT // 只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个 socket 的话，需要再次把这个 socket 加入到 EPOLL 队列里
```

### int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout)

man 参考：http://man7.org/linux/man-pages/man2/epoll_wait.2.html

等待事件的产生，类似于 select() 调用。

参数 events 用来从内核得到事件的集合，maxevents 告之内核这个 events 有多大，这个 maxevents 的值不能大于创建 epoll_create() 时的 size。

参数 timeout 是超时时间（毫秒，0 会立即返回，-1 将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回 0 表示已超时。

epoll_wait 范围之后应该是一个循环，遍历所有的事件。

## 示例代码

## select & poll & epoll 比较
