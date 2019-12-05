## 目录

- [消息队列](#消息队列)
- [IPC 对象数据结构](#IPC-对象数据结构)
- [消息队列数据结构](#消息队列数据结构)
- [消息队列函数](#消息队列函数)
- [](#)

## 消息队列

1. 消息队列提供了一个从一个进程向另外一个进程发送一块数据的方法
2. 每个数据块都被认为是有一个类型，接收者进程接收的数据块可以有不同的类型值
3. 消息队列与管道不同的是，消息队列是基于消息的，而管道是基于字节流的，且消息队列的读取不一定是先入先出。
4. 消息队列也有管道一样的不足，就是每个消息的最大长度是有上限的（MSGMAX），每个消息队列的总的字节数是有上限的（MSGMNB），系统上消息队列的总数也有一
个上限（MSGMNI），这三个参数都可以查看：

```shell
cjl@S405:~$ cat /proc/sys/kernel/msgmax  # 一条消息的长度
8192
cjl@S405:~$ cat /proc/sys/kernel/msgmnb  # 一个消息队列总的容纳长度
16384
cjl@S405:~$ cat /proc/sys/kernel/msgmni  # 系统中创建消息队列的最大个数
1663
```

## IPC 对象数据结构

内核为每个 IPC 对象维护一个数据结构，位于 `<sys/ipc.h>`。

```c
struct ipc_perm
{
    key_t           key;   /* 调用 shmget() 时给出的关键字 */
    uid_t           uid;   /* 共享内存所有者的有效用户 ID */
    gid_t           gid;   /* 共享内存所有者所属组的有效组 ID */
    uid_t           cuid;  /* 共享内存创建者的有效用户 ID */
    gid_t           cgid;  /* 共享内存创建者所属组的有效组 ID */
    unsigned short  mode;  /* Permissions + SHM_DEST 和 SHM_LOCKED 标志 */
    unsignedshort   seq;   /* 序列号 */
};
```

## 消息队列数据结构

定义在`<sys/msg.h>`。

```c
struct msqid_ds
{
    struct ipc_perm msg_perm;     /* Ownership and permissions */
    time_t          msg_stime;    /* Time of last msgsnd(2) */
    time_t          msg_rtime;    /* Time of last msgrcv(2) */
    time_t          msg_ctime;    /* Time of last change */
    unsigned long   __msg_cbytes; /* Current number of bytes in queue (nonstandard) */
    msgqnum_t       msg_qnum;     /* Current number of messages in queue */
    msglen_t        msg_qbytes;   /* Maximum number of bytes allowed in queue */
    pid_t           msg_lspid;    /* PID of last msgsnd(2) */
    pid_t           msg_lrpid;    /* PID of last msgrcv(2) */
};
```

消息队列是用链表实现的，这里需要提出的是 MSGMAX 指的是一条消息的数据大小的上限，所有消息数据总和不能超过 MSGMNB。像下面这幅图是一条消息队列，系统含有这样的条数总数不能超过 MSGMNI 个。

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/025.png)

## 消息队列函数

**（1）创建和访问一个消息队列**

```c
int msgget(key_t key,   // 消息队列的名字
           int msgflg); // 由个权限标志组成,它们的用法和创建文件时使用的 mode 模式标志时一样的
```

与其他的 IPC 机制一样，程序必须提供一个键来命名某个特定的消息队列。msgflg 是一个权限标志，表示消息队列的访问权限，它与文件的访问权限一样。msgflg 可以与 IPC_CREAT 做或操作，表示当 key 所命名的消息队列不存在时创建一个消息队列，如果 key 所命名的消息队列存在时，IPC_CREAT 标志会被忽略，而只返回一个标识符。

返回值：成功返回非负整数，即消息队列的标识码；失败返回 -1。

**（2）消息队列的控制函数**

```c
int msgctl(int msgid,              // msgget 函数返回的消息队列标识符
           int cmd,                // 采取的动作
           struct msgid_ds *buf);  //
```

返回值：成功返回 0；失败返回 -1。

cmd 取值如下，

- IPC_STAT：把 msqid_ds 结构中的数据设置为消息队列的当前关联值
- IPC_SET：在进程有足够权限的前提下，把消息队列的当前关联值设置为 msqid_ds 数据结构中给出的值
- IPC_RMID：删除消息队列

**（3）把一条消息添加到消息队列中**

```c
int msgsnd(int         msqid,   // 由 msgget 返回的消息队列标志吗
           const void *msgp,    // 准备发送的消息
           size_t      msgsz,   // 是 msgp 指向消息的长度
           int         msgflg); // 控制这当前消息队列满或到达系统上限时发生的事情
```

msgp 是一个指向准备发送消息的指针，但是消息的数据结构却有一定的要求，指针 msgp 所指向的消息结构一定要是以一个长整型成员变量开始的结构体，接收函数将用这个成员来确定消息的类型。所以消息结构要定义成这样： 

```c
struct my_message
{
    long int message_type;
    
    /* The data you wish to transfer */
};
```
msgsz 是 msgp 指向的消息的长度，注意是消息的长度，而不是整个结构体的长度，也就是说 msgsz 是不包括长整型消息类型成员变量的长度。

msgflg 用于控制当前消息队列满或队列消息到达系统范围的限制时将要发生的事情。msgflg = IPC_NOWAIT 表示队列满不等待，返回 EAGAIN 错误。为 0 表示阻塞等待。

返回值：成功返回 0；失败返回 -1。

**（4）从一个消息队列接收消息**

```c
 ssize_t msgrcv(int    msqid,   // 消息队列标识符
                void  *msgp,    // 指针指向准备接收的消息
                size_t msgsz,   // 消息长度
                long   msgtype,  // 实现接收优先级的简单形式
                int    msgflg); // 控制着队列中没有相应类型的消息可供接收时将要发生的事
```

msgid, msgp, msgsz 的作用也函数 msgsnd() 函数的一样。

msgtype 可以实现一种简单的接收优先级。如果 msgtype 为 0，就获取队列中的第一个消息。如果它的值大于零，将获取具有相同消息类型的第一个信息。如果它小于零，就获取类型等于或小于 msgtype 的绝对值的第一个消息。

msgflg 用于控制当队列中没有相应类型的消息可以接收时将发生的事情。

调用成功时，该函数返回放到接收缓存区中的字节数，消息被复制到由 msgp 指向的用户分配的缓存区中，然后删除消息队列中的对应消息；失败时返回 -1。




