## 打开，创建，关闭，销毁一个消息队列

```c
#include<fcntl.h>
#include<sys/stat.h>
#include<mqueue.h>

mqd_t mq_open(const char *name, int oflag);
mqd_t mq_open(const char *name, int oflag, mode_t mode, struct mq_attr *attr);
mqd_t mq_close(mqd_t mqdes);
mqd_t mq_unlink(const char *name);
```

name：表示消息队列的名字，它符合 POSIX IPC 的名字规则，最大长度为 NAME_MAX(255) 个字符。。

oflag：表示打开的方式，和 open 函数的类似。

|    标记    |                       描述                        |
| :--------: | :-----------------------------------------------: |
|  O_CREAT   |               队列不存在时创建队列                |
|   O_EXCL   | 与 O_CREAT 一起使用，若消息队列已存在，则错误返回 |
|  O_RDONLY  |                     只读打开                      |
|  O_WRONLY  |                     只写打开                      |
|   O_RDWR   |                     读写打开                      |
| O_NONBLOCK |                 以非阻塞模式打开                  |

oflag 参数的其中一个用途是，确定是打开一个既有队列还是创建和打开一个新队列。如果在 oflag 中不包含 O_CREAT，那么将会打开一个既有队列。如果在 oflag 中
包含了 O_CREAT，并且与给定的 name 对应的队列不存在，那么就会创建一个新的空队列。

如果在 oflag 中同时包含 O_CREAT 和 O_EXCL，并且与给定的 name 对应的队列已经存在，那么 mq_open() 就会失败。

mq_open() 通常用来打开一个既有消息队列，这种调用只需要两个参数，但如果在 oflags 中指定了 O_CREAT，那么就还需要另外两个参数：mode 和 attr。这些
参数用法如下：

mode：是一个位掩码，它指定了施加于新消息队列之上的权限。这个参数可取的位置与文件上的掩码值是一样的，并且与 open() 一样，mode 中的值会与进程
的 umask 取掩码。要从一个队列中读取消息就必须要将读权限赋予相应的用户，要向队列写入消息就需要写权限。

attr：是一个 mq_attr 结构，它指定了新消息队列的特性。如果 attr 为 NULL，那么将使用实现定义的默认特性创建队列。mq_attr 结构后在后面进行介绍。

mq_open 返回值是 mqd_t 类型的值（出错返回 -1），被称为消息队列描述符。在 Linux 2.6.18 中该类型的定义为整型：

```c
#include <bits/mqueue.h>
typedef int mqd_t;
```

mq_close 用于关闭一个消息队列，和文件的 close 类型，关闭后，消息队列并不从系统中删除。一个进程结束，会自动调用关闭打开着的消息队列。

mq_unlink 用于删除一个消息队列。消息队列创建后只有通过调用该函数或者是内核自举才能进行删除。每个消息队列都有一个保存当前打开着描述符数的引用计数器，
和文件一样，因此本函数能够实现类似于 unlink 函数删除一个文件的机制。

POSIX 消息队列的名字所创建的真正路径名和具体的系统实现有关，关于具体 POSIX IPC 的名字规则可以参考《UNIX 网络编程 卷2：进程间通信》的 P14。

经过测试，在 Linux 2.6.18 中，所创建的 POSIX 消息队列不会在文件系统中创建真正的路径名。且 POSIX 的名字只能以一个`/`开头，名字中不能包含其他的`/`。

## POSIX 消息队列的属性

```c
struct mq_attr
{
        long mq_flags;   // 消息队列的标志：0 或 O_NONBLOCK，用来表示是否阻塞
        long mq_maxmsg;  // 消息队列的最大消息数
        long mq_msgsize; // 消息队列中每个消息的最大字节数
        long mq_curmsgs; // 消息队列中当前的消息数
};
```











