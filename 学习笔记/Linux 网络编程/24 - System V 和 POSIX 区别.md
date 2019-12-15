## 目录

- [System V 和 POSIX 区别](#System-V-和-POSIX-区别)
- [消息队列](#消息队列)
- [共享内存](#共享内存)
- [信号量](#信号量)

## System V 和 POSIX 区别

System V， 曾经也被称为 AT&T System V，是 Unix 操作系统众多版本中的一支。它最初由 AT&T 开发，在 1983 年第一次发布。一共发行了 4 个 System V 的主要版本：版本 1、2、3 和 4。System V Release 4，或者称为 SVR4，是最成功的版本，成为一些 UNIX 共同特性的源头，例如 SysV 初始化脚本 /etc/init.d，用来控制系统启动和关闭，System V Interface Definition (SVID) 是一个 System V 如何工作的标准定义。AT&T 出售运行 System V 的专有硬件，但许多(或许是大多数)客户在其上运行一个转售的版本，这个版本基于 AT&T 的实现说明。流行的 SysV 衍生版本包括 Dell SVR4 和 Bull SVR4。当今广泛使用的 System V 版本是 SCO OpenServer，基于 System V Release 3，以及 SUN Solaris 和 SCO UnixWare，都基于 System V Release 4。System V 是 AT&T 的第一个商业UNIX版本(UNIX System III)的加强。传统上，System V 被看作是两种 UNIX 风味之一(另一个是 BSD)。然而，随着一些并不基于这两者代码的 UNIX 实现的出现，例如 Linux 和 QNX， 这一归纳不再准确，但不论如何，像 POSIX 这样的标准化努力一直在试图减少各种实现之间的不同。

POSIX(Portable Operating System Interface for Computing Systems)是由 IEEE 和 ISO/IEC 开发的一簇标准。该标准是基于现有的 UNIX 实践和经验，描述了操作系统的调用服务接口，用于保证编制的应用程序可以在源代码一级上在多种操作系统上移植运行。它是在 1980 年早期一个 UNIX 用户组(usr/group)的早期工作的基础上取得的。该 UNIX 用户组原来试图将 AT&T 的系统 V 和 Berkeley CSRG 的 BSD 系统的调用接口之间的区别重新调和集成，从而于 1984 年产生 /usr/group 标准。1985 年，IEEE 操作系统技术委员会标准小组委员会(TCOS-SS)开始在 ANSI 的支持下责成 IEEE 标准委员会制定有关程序源代码可移植性操作系统服务接口正式标准。到了 1986 年 4 月，IEEE 就制定出了试用标准。第一个正式标准是在 1988 年 9 月份批准的(IEEE 1003.1-1988)，也既以后经常提到的 POSIX.1 标准。

| SYSTEM V                                                     | POSIX                                                        |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1983 年 由 AT&T 开发的三种进程间通信：消息队列，共享内存和信号量。 | 由 IEEE 和 ISO/IEC 开发的一簇标准，同样包含了三种进程间通信。 |
| 包含所有 IPC机制：管道，命名管道，消息队列，信号，信号量和共享内存。同时也有 unix 域套接字。 | 和 System V 一样，只是借口不同而已。                         |
| 共享内存接口shmget(), shmat(), shmdt(), shmctl()             | 共享内存接口 shm_open(), mmap(), shm_unlink()                |
| 消息队列接口msgget(), msgsnd(), msgrcv(), msgctl()           | 消息队列接口 mq_open(), mq_send(), mq_receive(), mq_unlink() |
| 信号量接口 semget(), semop(), semctl()                       | 命名信号量 sem_open(), sem_close(), sem_unlink(), sem_post(), sem_wait(), sem_trywait(), sem_timedwait(), sem_getvalue()。无名或是基于内存的信号量 sem_init(), sem_post(), sem_wait(), sem_getvalue(),sem_destroy() |
| 使用 keys 和标识符来分辨IPC 对象                             | 使用名字和文件描述符来分辨 IPC 对象                          |
|                                                              | 消息队列可以被 select(), poll() and epoll 监控               |
| 提供 msgctl()                                                | 提供 mq_getattr() and mq_setattr()                           |
|                                                              | 多线程安全。包含线程同步（锁，条件变量，读写锁等等）         |
|                                                              | 消息队列有通知机制，比如接口 mq_notify()                     |
| 需要系统调用 shmctl() 或命令去控制或者状态检查。             | 共享内存可以通过 fstat(), fchmod() 来检查。                  |
| 共享内存段的大小在 shmget() 时就固定了。                     | 可以使用 ftruncate() 来调整大小，然后通过 munmap() 和 mmap() 重新创建。 |

参考：<https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_system_v_posix.htm>

stackoverflow 上探讨：https://stackoverflow.com/questions/4582968/system-v-ipc-vs-posix-ipc

## 消息队列

1. 对 posix 消息队列的读总是返回最高优先级的最早消息，对 system v 消息队列的读则可以返回任意指定优先级的消息
2. 当往一个空队列放置一个消息时，posix 消息队列允许产生一个信号或启动一个线程，system v 消息队列则不提供类似机制

## 共享内存

1. Posix 使用一个以 / 打头的名字来获取共享内存，在 /dev/shm 目录下可以看到创建的共享内存。而 System V 使用 key 来对应唯一的共享内存标示符，为了避免不必要的麻烦，我们使用 ftok 函数来生成 key。可以使用命令 ipcs 查看创建的 System V 共享内存。
2. Posix使用 mmap 函数来将共享内存映射到进程空间，而 System V 使用函数 shmat，不同的是 mmap 还可以将文件映射到进程空间。
3. Posix 的共享内存引用变为 0 的时候就自动删除，而 System V 即使连接数为 0 也不会自动删除，只能调用函数 shmctl 来手动删除。

## 信号量






























