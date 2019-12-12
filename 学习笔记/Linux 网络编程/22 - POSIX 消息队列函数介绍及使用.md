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

POSIX 消息队列的属性设置和获取可以通过下面两个函数实现：

```c
#include <mqueue.h>
mqd_t mq_getattr(mqd_t mqdes, struct mq_attr *attr);
mqd_t mq_setattr(mqd_t mqdes, struct mq_attr *newattr, struct mq_attr *oldattr);
```

mq_getattr 用于获取当前消息队列的属性，mq_setattr 用于设置当前消息队列的属性。其中 mq_setattr 中的 oldattr 用于保存修改前的消息队列的属性，可以为空。

mq_setattr 可以设置的属性只有 mq_flags，用来设置或清除消息队列的非阻塞标志。newattr 结构的其他属性被忽略。mq_maxmsg 和 mq_msgsize 属性只能在创建消息队列时通过 mq_open 来设置。mq_open 只会设置该两个属性，忽略另外两个属性。mq_curmsgs 属性只能被获取而不能被设置。

```c++
#include <iostream>
#include <cstring>
 
#include <errno.h>
#include <unistd.h>
#include <fcntl.h>
#include <mqueue.h>
 
using namespace std;
 
int main()
{
    mqd_t mqID;
    mqID = mq_open("/anonymQueue", O_RDWR | O_CREAT, 0666, NULL);
 
    if (mqID < 0)
    {
        cout<<"open message queue error..."<<strerror(errno)<<endl;
        return -1;
    }
 
    mq_attr mqAttr;
    if (mq_getattr(mqID, &mqAttr) < 0)
    {
        cout<<"get the message queue attribute error"<<endl;
        return -1;
    }
 
    cout<<"mq_flags:"<<mqAttr.mq_flags<<endl;
    cout<<"mq_maxmsg:"<<mqAttr.mq_maxmsg<<endl;
    cout<<"mq_msgsize:"<<mqAttr.mq_msgsize<<endl;
    cout<<"mq_curmsgs:"<<mqAttr.mq_curmsgs<<endl;
}
```

在 Linux 2.6.18 中执行结果是：

```
mq_flags:0
mq_maxmsg:10
mq_msgsize:8192
mq_curmsgs:0
```

## 发送消息和接收消息

```c
#include <mqueue.h>

int mq_send(mqd_t mqdes,const char *msg_ptr,size_t msg_len,unsigned int msg_prio);
ssize_t mq_receive(mqd_t mqdes, char *mdg_ptr,size_t msg_len,unsigned int *msg_prio);


#ifdef __USE_XOPEN2K
mqd_t mq_timedsend(mqd_t mqdes, const char *msg_ptr,
                      size_t msg_len, unsigned msg_prio,
                      const struct timespec *abs_timeout);
 
mqd_t mq_timedreceive(mqd_t mqdes, char *msg_ptr,
                      size_t msg_len, unsigned *msg_prio,
                      const struct timespec *abs_timeout);
#endif
```

mq_send 向消息队列中写入一条消息，mq_receive 从消息队列中读取一条消息。

mqdes：消息队列描述符；

msg_ptr：指向消息体缓冲区的指针；

msg_len：消息体的长度，其中 mq_receive 的该参数不能小于能写入队列中消息的最大值，即一定要大于等于该队列的 mq_attr 结构中 mq_msgsize 的大小。如果mq_receive 中的 msg_len 小于该值，就会返回 EMSGSIZE 错误。POXIS 消息队列发送的消息长度可以为 0。

msg_prio：消息的优先级；它是一个小于 MQ_PRIO_MAX 的数，数值越大，优先级越高。POSIX 消息队列在调用 mq_receive 时总是返回队列中最高优先级的最早消息。如果消息不需要设定优先级，那么可以在 mq_send 是置 msg_prio 为 0，mq_receive 的 msg_prio 置为 NULL。

默认情况下 mq_send 和 mq_receive 是阻塞进行调用，可以通过 mq_setattr 来设置为 O_NONBLOCK。

```c++
#include <iostream>
#include <cstring>
#include <errno.h>
 
#include <unistd.h>
#include <fcntl.h>
#include <mqueue.h>
 
using namespace std;
 
int main()
{
    mqd_t mqID;
    mqID = mq_open("/anonymQueue", O_RDWR | O_CREAT | O_EXCL, 0666, NULL);
 
    if (mqID < 0)
    {
        if (errno == EEXIST)
        {
            mq_unlink("/anonymQueue");
            mqID = mq_open("/anonymQueue", O_RDWR | O_CREAT, 0666, NULL);
        }
        else
        {
            cout<<"open message queue error..."<<strerror(errno)<<endl;
            return -1;
        }
    }
 
    if (fork() == 0)
    {
        mq_attr mqAttr;
        mq_getattr(mqID, &mqAttr);
 
        char *buf = new char[mqAttr.mq_msgsize];
 
        for (int i = 1; i <= 5; ++i)
        {
            if (mq_receive(mqID, buf, mqAttr.mq_msgsize, NULL) < 0)
            {
                cout<<"receive message  failed. ";
                cout<<"error info:"<<strerror(errno)<<endl;
                continue;
            }
 
            cout<<"receive message "<<i<<": "<<buf<<endl;   
        }
        exit(0);
    }
 
    char msg[] = "yuki";
    for (int i = 1; i <= 5; ++i)
    {
        if (mq_send(mqID, msg, sizeof(msg), i) < 0)
        {
            cout<<"send message "<<i<<" failed. ";
            cout<<"error info:"<<strerror(errno)<<endl;
        }
        cout<<"send message "<<i<<" success. "<<endl;   
 
        sleep(1);
    }
}                     
```

在 Linux 2.6.18 下的执行结构如下：

```
send message 1 success. 
receive message 1: yuki
send message 2 success. 
receive message 2: yuki
send message 3 success. 
receive message 3: yuki
send message 4 success. 
receive message 4: yuki
send message 5 success. 
receive message 5: yuki
```

## POSIX 消息队列的限制

POSIX 消息队列本身的限制就是 mq_attr 中的 mq_maxmsg 和 mq_msgsize，分别用于限定消息队列中的最大消息数和每个消息的最大字节数。在前面已经说过了，这两个参数可以在调用 mq_open 创建一个消息队列的时候设定。当这个设定是受到系统内核限制的。

下面是在 Linux 2.6.18 下 shell 对启动进程的 POSIX 消息队列大小的限制：

```
# ulimit -a |grep message
POSIX message queues     (bytes, -q) 819200
```

限制大小为 800KB，该大小是整个消息队列的大小，不仅仅是最大消息数消息的最大大小；还包括消息队列的额外开销。前面我们知道 Linux 2.6.18 下 POSIX 消息队列默认的最大消息数和消息的最大大小分别为：

```
mq_maxmsg = 10
mq_msgsize = 8192
```

为了说明上面的限制大小包括消息队列的额外开销，下面是测试代码：

```c++
#include <iostream>
#include <cstring>
#include <errno.h>
 
#include <unistd.h>
#include <fcntl.h>
#include <mqueue.h>
 
using namespace std;
 
int main(int argc, char **argv)
{
    mqd_t mqID;
    mq_attr attr;
    attr.mq_maxmsg = atoi(argv[1]);
    attr.mq_msgsize = atoi(argv[2]);
 
    mqID = mq_open("/anonymQueue", O_RDWR | O_CREAT | O_EXCL, 0666, &attr);
 
    if (mqID < 0)
    {
        if (errno == EEXIST)
        {
            mq_unlink("/anonymQueue");
            mqID = mq_open("/anonymQueue", O_RDWR | O_CREAT, 0666, &attr);
 
            if(mqID < 0)
            {
                cout<<"open message queue error..."<<strerror(errno)<<endl;
                return -1;
            }
        }
        else
        {
            cout<<"open message queue error..."<<strerror(errno)<<endl;
            return -1;
        }
    }
 
    mq_attr mqAttr;
    if (mq_getattr(mqID, &mqAttr) < 0)
    {
        cout<<"get the message queue attribute error"<<endl;
        return -1;
    }
 
    cout<<"mq_flags:"<<mqAttr.mq_flags<<endl;
    cout<<"mq_maxmsg:"<<mqAttr.mq_maxmsg<<endl;
    cout<<"mq_msgsize:"<<mqAttr.mq_msgsize<<endl;
    cout<<"mq_curmsgs:"<<mqAttr.mq_curmsgs<<endl; 
}
```

下面进行创建消息队列时设置最大消息数和消息的最大大小进行测试：

```shell
[root@idcserver program]# g++ -g test.cpp -lrt
[root@idcserver program]# ./a.out 10 81920
open message queue error...Cannot allocate memory
[root@idcserver program]# ./a.out 10 80000
open message queue error...Cannot allocate memory
[root@idcserver program]# ./a.out 10 70000
open message queue error...Cannot allocate memory
[root@idcserver program]# ./a.out 10 60000
mq_flags:0
mq_maxmsg:10
mq_msgsize:60000
mq_curmsgs:0
```

从上面可以看出消息队列真正存放消息数据的大小是没有 819200B 的。可以通过修改该限制参数，来改变消息队列的所能容纳消息的数量。可以通过下面方式来修改限制，但这会在 shell 启动进程结束后失效，可以将设置写入开机启动的脚本中执行，例如 .bashrc，rc.local。

```shell
[root@idcserver ~]# ulimit -q 1024000000
[root@idcserver ~]# ulimit -a |grep message
POSIX message queues     (bytes, -q) 1024000000
```

下面再次测试可以设置的消息队列的属性。

```shell
[root@idcserver program]# ./a.out 10 81920
mq_flags:0
mq_maxmsg:10
mq_msgsize:81920
mq_curmsgs:0
[root@idcserver program]# ./a.out 10 819200
mq_flags:0
mq_maxmsg:10
mq_msgsize:819200
mq_curmsgs:0
[root@idcserver program]# ./a.out 1000 8192  
mq_flags:0
mq_maxmsg:1000
mq_msgsize:8192
mq_curmsgs:0
```

POSIX 消息队列在实现上还有另外两个限制：

- MQ_OPEN_MAX：一个进程能同时打开的消息队列的最大数目，POSIX 要求至少为 8；
- MQ_PRIO_MAX：消息的最大优先级，POSIX 要求至少为 32；


