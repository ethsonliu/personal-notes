## 目录

- [消息队列](#消息队列)
- [IPC 对象数据结构](#IPC-对象数据结构)
- [消息队列数据结构](#消息队列数据结构)
- [](#)
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

消息队列是用链表实现的，这里需要提出的是 MSGMAX 指的是一条消息的纯数据大小的上限，下图是一个消息队列，则其纯数据总和不能超过 MSGMNB，像这样一条消息队列，系统含有的总数不能超过 MSGMNI 个。

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/025.png)





