## 目录

- [共享内存](#共享内存)
- [管道-消息队列-共享内存传递数据对比](#管道-消息队列-共享内存传递数据对比)
- [mmap 函数](#mmap-函数)
- [共享内存数据结构](#共享内存数据结构)
- [共享内存 API](#共享内存-API)
- [示例](#示例)

## 共享内存

共享内存区是最快的 IPC 形式。一旦这样的内存映射到共享它的进程的地址空间，这些进程间数据传递不再涉及到内核，换句话说是进程不再通过执行进入内核的系统
调用来传递彼此的数据。

## 管道-消息队列-共享内存传递数据对比

**匿名管道**

1. 优点，不需要加锁
2. 缺点，默认缓冲区太小，只有 4k
3. 进程父子间通信
4. 单向通信，半双工，通信时需要关闭不需要的读写
5. 是一种非永久性的管道通信机构，当它访问的进程全部终止时，它也将随之被撤消

**命名管道(一个文件)**

1. 优点，不需要加锁
2. 缺点，默认缓冲区太小，只有 4k
3. 可以多进程通信
4. 单向通信
5. FIFO 是一种永久的管道通信机构。通信完毕后，可使用 close() 将管道文件关闭。命名管道的文件是硬盘上的设备文件，是可见的。
6. 不同于匿名管道之处在于它提供一个路径名与之关联，以 FIFO 的文件形式存在于文件系统中。这样，即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可
以访问该路径，就能够彼此通过 FIFO 相互通信。

**消息队列**

1. 不需要加锁
2. 默认缓冲区和单消息上限都要大一些
3. 不局限于父子进程间通信
4. 也可以双向通信不过稍微加个标识，可以通过消息中的type进行区分，比如一个任务分派进程，创建了若干个执行子进程，不管是父进程发送分派任务的消息，还是子进程发送任务执行的消息，都
将 type 设置为目标进程的 pid，因为 msgrcv 可以指定只接收消息类型为 type 的消息，这样就实现了子进程只接收自己的任务，父进程只接收任务结果
5. 只需要相同的 key，就可以让不同进程定位到同一消息队列上
6. 与命名管道相比：消息队列的优势在于，它独立于发送和接收进程而存在，这消除了在同步命名管道的打开和关闭时可能产生的一些困难。消息队列
提供了一种从一个进程向另一个进程发送一个数据块的方法。而且，每个数据块被认为含有一个类型，接收进程可以独立地接收含有不同类型值的数据块。
7. 优点，我们可以通过发送消息来几乎完全避免命名管道的同步和阻塞问题。我们可以用一些方法来提前查看紧急消息。
8. 缺点，与管道一样，每个数据块有一个最大长度的限制。系统中所有队列所包含的全部数据块的总长度也有一个上限。
9. Linux 系统中有两个宏定义:MSGMAX, 以字节为单位，定义了一条消息的最大长度。MSGMNB, 以字节为单位，定义了一个队列的最大长度。

**共享内存**

1. 没有上限
2. 不局限于父子进程，采用跟消息队列类似的定位方式
3. 不存在任何单向的限制
4. 需要应用程序自己做互斥( 若一个进程正在向共享内存区写数据，则在它做完这一步操作前，别的进程不应当去读、写这些数据。) 有如下三种互斥方案：
    1. 只适用两个进程共享，在内存中放一个标志位，一定要声明为 volatile，大家基于标志位来互斥，例如为0时第一个可以写，第二个就等待，为 1 时第一个等待，第二个可以写/读
    2. 也只适用两个进程，是用信号。
    3. 采用信号量或者 msgctl 自己的加锁、解锁功能，不过后者只适用于 linux

消息队列，FIFO，管道的消息传递方式一般为，

1. 服务器得到输入
2. 通过管道，消息队列写入数据，通常需要从进程拷贝到内核。
3. 客户从内核拷贝到进程
4. 然后再从进程中拷贝到输出文件

上述过程通常要经过 4 次拷贝，才能完成文件的传递。

共享内存只需要，

1. 从输入文件到共享内存区
2. 从共享内存区输出到文件

上述过程不涉及到内核的拷贝，所以花的时间较少。

## mmap 函数

**将文件或者设备空间映射到共享内存区**

```c
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);
```

成功执行时，mmap() 返回被映射区的指针。失败时，mmap() 返回 MAP_FAILED ，其值为`(void*)-1`， error 被设为以下的某个值：

```
EACCES： 访问出错
EAGAIN： 文件已被锁定，或者太多的内存已被锁定
EBADF：  fd 不是有效的文件描述词
EINVAL： 一个或者多个参数无效
ENFILE： 已达到系统对打开文件的限制
ENODEV： 指定文件所在的文件系统不支持内存映射
ENOMEM： 内存不足，或者进程已超出最大内存映射数量
EPERM：  权能不足，操作不允许
ETXTBSY：已写的方式打开文件，同时指定 MAP_DENYWRITE 标志
SIGSEGV：试着向只读区写入
SIGBUS： 试着访问不属于进程的内存区
```

参数解释如下，

start：映射区的开始地址

length：映射区的长度

prot：期望的内存保护标志，不能与文件的打开模式冲突。是以下的某个值，可以通过 or 运算合理地组合在一起

```
PROT_EXEC： 页内容可以被执行
PROT_READ： 页内容可以被读取
PROT_WRITE：页可以被写入
PROT_NONE： 页不可访问
```

flags：指定映射对象的类型，映射选项和映射页是否可以共享。它的值可以是一个或者多个以下位的组合体

```
MAP_FIXED      // 使用指定的映射起始地址，如果由start和 len 参数指定的内存区重叠于现存的映射空间，重叠部分将会被丢弃。如果指定的起始地址不可用，操作将会失败。并且起始地址必须落在页的边界上。
MAP_SHARED     // 与其它所有映射这个对象的进程共享映射空间。对共享区的写入，相当于输出到文件。直到 msync() 或者 munmap() 被调用，文件实际上不会被更新。
MAP_PRIVATE    // 建立一个写入时拷贝的私有映射。内存区域的写入不会影响到原文件。这个标志和以上标志是互斥的，只能使用其中一个。
MAP_DENYWRITE  // 这个标志被忽略。
MAP_EXECUTABLE // 同上
MAP_NORESERVE  // 不要为这个映射保留交换空间。当交换空间被保留，对映射区修改的可能会得到保证。当交换空间不被保留，同时内存不足，对映射区的修改会引起段违例信号。
MAP_LOCKED     // 锁定映射区的页面，从而防止页面被交换出内存。
MAP_GROWSDOWN  // 用于堆栈，告诉内核VM系统，映射区可以向下扩展。
MAP_ANONYMOUS  // 匿名映射，映射区不与任何文件关联。
MAP_ANON       // MAP_ANONYMOUS 的别称，不再被使用。
MAP_FILE       // 兼容标志，被忽略。
MAP_32BIT      // 将映射区放在进程地址空间的低 2GB，MAP_FIXED 指定时会被忽略。当前这个标志只在 x86-64 平台上得到支持。
MAP_POPULATE   // 为文件映射通过预读的方式准备好页表。随后对映射区的访问不会被页违例阻塞。
MAP_NONBLOCK   // 仅和 MAP_POPULATE 一起使用时才有意义。不执行预读，只为已存在于内存中的页面建立页表入口。
```

fd：有效的文件描述词。如果 MAP_ANONYMOUS 被设定，为了兼容问题，其值应为 -1

offset：被映射对象内容的起点

注意点：

1. 映射不能改变文件的大小
2. 可用于进程间通信的有效地址空间不完全受限于被映射文件的大小
3. 文件一旦被映射后，所有对映射区域的访问实际上是对内存区域的访问。映射区域内容写回文件时，所写内容不能超过文件的大小

下图为内存映射文件示意图，

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/026.png)

其它需要使用的函数，

```c
int munmap(void *addr, size_t len);
```

成功执行时，munmap() 返回 0。失败时，munmap 返回-1，error 返回标志和 mmap 一致；

该调用在进程地址空间中解除一个映射关系，addr 是调用 mmap() 时返回的地址，len 是映射区的大小；当映射关系解除后，对原来映射地址的访问将导致段错误发生。 

```c
int msync(void *addr, size_t len, int flags);
```

一般说来，进程在映射空间的对共享内容的改变并不直接写回到磁盘文件中，往往在调用 munmap() 后才执行该操作。可以通过调用 msync() 实现磁盘上文件内容与共享内存区的内容一致。

flags 的值可以为，MS_ASYNC 执行异步写；MS_SYNC 执行同步写；MS_INVALIDATE 使高速缓存的数据失效

参考：<https://www.cnblogs.com/huxiao-tee/p/4660352.html>

下边是读取文件的示例：

```c++

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>

typedef struct{
  char name[20];
  short age;
  float score;
  char sex;
}student;

int main()
{
  student *p,*pend;  
  
  //打开文件描述符号
  int fd;
  
   /*打开文件*/
    fd=open("user.dat",O_RDWR);
    if(fd==-1) //文件不存在
    {
        fd=open("user.dat",O_RDWR|O_CREAT,0666);
        if(fd==-1){
            printf("打开或创建文件失败:%m\n");
            exit(-1);
        }
    }
    
  //打开文件ok，可以进行下一步操作
  printf("open ok!\n");  
  
  //获取文件的大小，映射一块和文件大小一样的内存空间，如果文件比较大，可以分多次，一边处理一边映射；
  struct stat st; //定义文件信息结构体
  
  /*取得文件大小*/
  int r=fstat(fd,&st);
  if(r==-1){
      printf("获取文件大小失败:%m\n");
      close(fd);
      exit(-1);
  }
  int len=st.st_size;    
  
  /*把文件映射成虚拟内存地址*/
  p=mmap(NULL,len,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);    
  if(p==NULL || p==(void*)-1){
      printf("映射失败:%m\n");
      close(fd);
      exit(-1);
  }
  
  /*定位到文件开始*/
  pend=p; 
  
  /*通过内存读取记录*/
  int i=0;
  while(i<(len/sizeof(student)))
  {
    printf("第%d个条\n",i);
    printf("name=%s\n",p[i].name);
    printf("age=%d\n",p[i].age);
    printf("score=%f\n",p[i].score);
    printf("sex=%c\n",p[i].sex);
    i++;
  }  
  
  /*卸载映射*/
  munmap(p,len);
  
  /*关闭文件*/    
  close(fd);    
}
```

## 共享内存数据结构

```c
struct shmid_ds {
    struct ipc_perm shm_perm;    /* Ownership and permissions */
    size_t          shm_segsz;   /* Size of segment (bytes) */
    time_t          shm_atime;   /* Last attach time */
    time_t          shm_dtime;   /* Last detach time */
    time_t          shm_ctime;   /* Last change time */
    pid_t           shm_cpid;    /* PID of creator */
    pid_t           shm_lpid;    /* PID of last shmat(2)/shmdt(2) */
    shmatt_t        shm_nattch;  /* No. of current attaches */
    ...
};
```

## 共享内存 API

```c
#include <sys/ipc.h>  
#include <sys/shm.h>  
  
int shmget(key_t key, size_t size, int shmflg);
void *shmat(int shmid, const void *shmaddr, int shmflg);
int shmdt(const void *shmaddr);
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

**创建共享内存,并将该内存的内容初始化为 0**

```c
int shmget(key_t key, size_t size, int shmflg);
```

打开一个已经存在共享内存, 如果打开时不知道共享内存的大小, 可以将 size 指定为0, shmflg 可以指定为 0(按照默认的权限打开);    

- key: 这个共享内存段名字;
- size: 共享内存大小(bytes);
- shmflg: 用法类似 msgget 中的 msgflg 参数;

返回值: 成功返回一个非负整数，即该共享内存段的标识码；失败返回 -1。

```c++
/**示例: 创建并打开一个共享内存 **/  
int main(int argc,char **argv)  
{  
    const int SHM_SIZE = 1024;  
    int shmid = shmget(0x1234, SHM_SIZE, 0666|IPC_CREAT);  
    if (shmid == -1)  
        err_exit("shmget error");  
    cout << "share memory get success" << endl;  
}  
```

**连接到本进程地址空间**

```c
void *shmat(int shmid, const void *shmaddr, int shmflg);  
```

at = attach，连接到本进程地址空间, 成功连接之后, 对该内存的操作就与 malloc 来的一块内存非常类似了, 而且如果这块内存中有数据, 则就可以直接将其中的数据取出来。

- shmaddr: 指定连接的地址(推荐使用 NULL)
- shmflg: 一般指定为 0, 表示可读,可写; 而它的另外两个可能取值是 SHM_RND 和 SHM_RDONLY(见下)

返回值： 成功返回一个指针，指向共享内存起始地址；失败返回`(void*)-1`。

| | |
|  :----  | :----  |
| shmaddr 为 NULL | Linux内核自动为进程连接到进程的内存(推荐使用) |
| shmaddr 不为 NULL且 shmflg 无 SHM_RND 标记  | 以 shmaddr 为连接地址 |
| shmflg = SHM_RDONLY  | 只读共享内存, 不然的话就是可读,可写的, 注意: 此处没有可读,可写这个概念 |

**将共享内存从当前进程中分离**

```c
int shmdt(const void *shmaddr);
```

dt = detach，注意，将共享内存分离并不是删除它，只是使该共享内存对当前进程不再可用。

- shmaddr: 由 shmat 所返回的指针

调用成功时返回 0，失败时返回 -1。

```c++
/** 示例: 将数据写入/读出共享内存 
程序write: 将数据写入共享内存 
程序read: 将数据读出共享内存(当然, 可以读取N多次) 
**/  
//write程序  
struct Student  
{  
    char name[26];  
    int age;  
};  
int main(int argc,char **argv)  
{  
    int shmid = shmget(0x1234, sizeof(Student), 0666|IPC_CREAT);  
    if (shmid == -1)  
        err_exit("shmget error");  
  
    // 以可读, 可写的方式连接该共享内存  
    Student *p = (Student *)shmat(shmid, NULL, 0);  
    if (p == (void *)-1)  
        err_exit("shmat error");  
    strcpy(p->name, "xiaofang");  
    p->age = 22;  
    shmdt(p);  
}  
```

```c++
//read程序  
int main(int argc,char **argv)  
{  
    int shmid = shmget(0x1234, 0, 0);  
    if (shmid == -1)  
        err_exit("shmget error");  
  
    // 以只读方式连接该共享内存  
    Student *p = (Student *)shmat(shmid, NULL, 0);  
    if (p == (void *)-1)  
        err_exit("shmat error");  
  
    // 直接将其中的内容打印输出  
    cout << "name: " << p->name << ", age: " << p->age << endl;  
    shmdt(p);  
}  
```

**设置/获取共享内存属性**

```c
int shmctl(int shm_id, int command, struct shmid_ds *buf);
```

- cmd: 将要采取的动作(三个取值见下)
- buf: 指向一个保存着共享内存的模式状态和访问权限的数据结构

```
IPC_STAT：把 shmid_ds 结构中的数据设置为共享内存的当前关联值，即用共享内存的当前关联值覆盖 shmid_ds 的值。
IPC_SET： 如果进程有足够的权限，就把共享内存的当前关联值设置为 shmid_ds 结构中给出的值
IPC_RMID：删除共享内存段
```

第三个参数，buf 是一个结构指针，它指向共享内存模式和访问权限的结构。

shmid_ds 结构至少包括以下成员：

```c
struct shmid_ds
{
    uid_t shm_perm.uid;
    uid_t shm_perm.gid;
    mode_t shm_perm.mode;
};
```

注意点，

1. 共享内存被别的程序占用,则删除该共享内存时,不会马上删除(引用计数计数);
2. 此时会出现一个现象:该共享内存的 key 变为 0x00000000,变为私有;
3. 此时还可以读,但必须还有办法获取该共享内存的 ID(shmid),因为此时试图通过该共享内存的 key 获取该共享内存,是白费的!

## 示例

```c++
// shmdata.h
#ifndef _SHMDATA_H_HEADER
#define _SHMDATA_H_HEADER
 
#define TEXT_SZ 2048
 
struct shared_use_st
{
    int written; // 作为一个标志，非0：表示可读，0：表示可写
    char text[TEXT_SZ]; // 记录写入 和 读取 的文本
};
 
#endif
```

```c++
// shmread.c
#include <stddef.h>
#include <sys/shm.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include "shmdata.h"
 
int main(int argc, char **argv)
{
    void *shm = NULL;
    struct shared_use_st *shared; // 指向shm
    int shmid; // 共享内存标识符
 
    // 创建共享内存
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
    if (shmid == -1)
    {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
 
    // 将共享内存连接到当前进程的地址空间
    shm = shmat(shmid, 0, 0);
    if (shm == (void *)-1)
    {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
 
    printf("\nMemory attached at %X\n", (int)shm);
 
    // 设置共享内存
    shared = (struct shared_use_st*)shm; // 注意：shm有点类似通过 malloc() 获取到的内存，所以这里需要做个 类型强制转换
    shared->written = 0;
    while (1) // 读取共享内存中的数据
    {
        // 没有进程向内存写数据，有数据可读取
        if (shared->written == 1)
        {
            printf("You wrote: %s", shared->text);
            sleep(1);
 
            // 读取完数据，设置written使共享内存段可写
            shared->written = 0;
 
            // 输入了 end，退出循环（程序）
            if (strncmp(shared->text, "end", 3) == 0)
            {
                break;
            }
        }
        else // 有其他进程在写数据，不能读取数据
        {
            sleep(1);
        }
    }
 
    // 把共享内存从当前进程中分离
    if (shmdt(shm) == -1)
    {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }
 
    // 删除共享内存
    if (shmctl(shmid, IPC_RMID, 0) == -1)
    {
        fprintf(stderr, "shmctl(IPC_RMID) failed\n");
        exit(EXIT_FAILURE);
    }
 
    exit(EXIT_SUCCESS);
}
```

```c++
// shmwrite.c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>
#include "shmdata.h"
 
int main(int argc, char **argv)
{
    void *shm = NULL;
    struct shared_use_st *shared = NULL;
    char buffer[BUFSIZ + 1]; // 用于保存输入的文本
    int shmid;
 
    // 创建共享内存
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
    if (shmid == -1)
    {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }
 
    // 将共享内存连接到当前的进程地址空间
    shm = shmat(shmid, (void *)0, 0);
    if (shm == (void *)-1)
    {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
 
    printf("Memory attched at %X\n", (int)shm);
 
    // 设置共享内存
    shared = (struct shared_use_st *)shm;
    while (1) // 向共享内存中写数据
    {
        // 数据还没有被读取，则等待数据被读取，不能向共享内存中写入文本
        while (shared->written == 1)
        {
            sleep(1);
            printf("Waiting...\n");
        }
 
        // 向共享内存中写入数据
        printf("Enter some text: ");
        fgets(buffer, BUFSIZ, stdin);
        strncpy(shared->text, buffer, TEXT_SZ);
 
        // 写完数据，设置written使共享内存段可读
        shared->written = 1;
 
        // 输入了end，退出循环（程序）
        if (strncmp(buffer, "end", 3) == 0)
        {
            break;
        }
    }
 
    // 把共享内存从当前进程中分离
    if (shmdt(shm) == -1)
    {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }
 
    sleep(2);
    exit(EXIT_SUCCESS);
}
```

分析：

1. 程序 shmread 创建共享内存，然后将它连接到自己的地址空间。在共享内存的开始处使用了一个结构 struct_use_st。该结构中有个标志 written，当共享内存中有其他进程向它写入数据时，共享内存中的 written 被设置为 0，程序等待。当它不为 0 时，表示没有进程对共享内存写入数据，程序就从共享内存中读取数据并输出，然后重置设置共享内存中的 written 为 0，即让其可被 shmwrite 进程写入数据。
2. 程序 shmwrite 取得共享内存并连接到自己的地址空间中。检查共享内存中的 written，是否为 0，若不是，表示共享内存中的数据还没有被完，则等待其他进程读取完成，并提示用户等待。若共享内存的 written 为 0，表示没有其他进程对共享内存进行读取，则提示用户输入文本，并再次设置共享内存中的 written 为 1，表示写完成，其他进程可对共享内存进行读操作。

关于前面的例子的安全性讨论:

这个程序是不安全的，当有多个程序同时向共享内存中读写数据时，问题就会出现。可能你会认为，可以改变一下 written 的使用方式，例如，只有当 written 为 0 时进程才可以向共享内存写入数据，而当一个进程只有在 written 不为 0 时才能对其进行读取，同时把 written 进行加 1 操作，读取完后进行减 1 操作。这就有点像文件锁中的读写锁的功能。咋看之下，它似乎能行得通。但是这都不是原子操作，所以这种做法是行不能的。试想当 written 为 0 时，如果有两个进程同时访问共享内存，它们就会发现 written 为 0，于是两个进程都对其进行写操作，显然不行。当 written 为 1 时，有两个进程同时对共享内存进行读操作时也是如些，当这两个进程都读取完是，written 就变成了 -1。

要想让程序安全地执行，就要有一种进程同步的进制，保证在进入临界区的操作是原子操作。例如，可以使用前面所讲的信号量来进行进程的同步。因为信号量的操作都是原子性的。

使用共享内存的优缺点：

1. 优点：我们可以看到使用共享内存进行进程间的通信真的是非常方便，而且函数的接口也简单，数据的共享还使进程间的数据不用传送，而是直接访问内存，也加快了程序的效率。同时，它也不像匿名管道那样要求通信的进程有一定的父子关系。
2. 缺点：共享内存没有提供同步的机制，这使得我们在使用共享内存进行进程间通信时，往往要借助其他的手段来进行进程间的同步工作。
