## 目录

- [信号量简介](#信号量简介)
- [相关函数](#相关函数)
- [示例](#示例)

## 信号量简介

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行进程访问代码的临
界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个进程在访问它，也就是说信
号量是用来调协进程对共享资源的访问的。

计数信号量具备两种操作动作，称为 V（signal()）与 P（wait()）（即部分参考书常称的“PV操作”）。V 操作会增加信号标 S 的数值，P 操作会减少它。

运作方式：

- 初始化，给与它一个非负数的整数值。
- 运行 P（wait()），信号标 S 的值将被减少。企图进入临界区段的进程，需要先运行 P（wait()）。当信号标 S 减为负值时，进程会被挡住，不能继续；当信号标 S 不为负值时，进程可以获准进入临界区段。
- 运行 V（signal()），信号标 S 的值会被增加。结束离开临界区段的进程，将会运行 V（signal()）。当信号标 S 不为负值时，先前被挡住的其他进程，将可获准进入临界区段。

可以这样理解，信号量就相当于是一个计数器。当有进程对它所管理的资源进行请求时，进程先要读取信号量的值，大于 0，资源可以请求，等于 0，资源不可以用，这时进程会进入睡眠状态直至资源可用。当一个进程不再使用资源时，信号量 +1(对应的操作称为 V 操作)，反之当有进程使用资源时，信号量 -1(对应的操作为P操作)。对信号量的值操作均为原子操作。

## 相关函数

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semget(key_t key, int nsems, int semflg);
int semctl(int semid, int semnum, int cmd, ...);
int semop(int semid, struct sembuf *sops, unsigned nsops);
```

**创建/访问一个信号量集**

```c
int semget(key_t key, int nsems, int semflg);
```

key: 信号集键(key)

nsems: 信号集中信号量的个数

semflg: 由九个权限标志构成，它们的用法和创建文件时使用的 mode 模式标志一致

返回值：成功返回一个非负整数，即该信号集的标识码；失败返回 -1

此时创建的信号量集中的每一个信号量都会有一个默认值: 0, 如果需要更改该值, 则需要调用 semctl 函数更改初始值;

```c++
/** 示例1: 封装一个创建一个信号量集函数  
该信号量集包含1个信号量; 
权限为0666 
**/  
int sem_create(key_t key)
{
    int semid = semget(key, 1, IPC_CREAT|IPC_EXCL|0666);
    if (semid == -1)
        err_exit("sem_create error");
    return semid;
}

/** 示例2: 打开一个信号量集 
nsems(信号量数量)可以填0, 
semflg(信号量权限)也可以填0, 表示使用默认的权限打开 
**/  
int sem_open(key_t key)  
{  
    int semid = semget(key, 0, 0);  
    if (semid == -1)  
        err_exit("sem_open error");  
    return semid;  
}  
```

**控制信号量集**

```c
int semctl(int semid, int semnum, int cmd, ...);
```

数

semid: 由 semget 返回的信号集标识码

semnum: 信号集中信号量的序号(注意: 从 0 开始)

cmd: 将要采取的动作(常用取值如下)

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/027.png)

如果该函数需要第四个参数(有时是不需要第四个参数的, 取决于 cmd 的取值), 则程序中必须定义如下的联合体:

```c
union semun  
{  
    int              val;    /* Value for SETVAL */  
    struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET ，介绍见下面*/  
    unsigned short  *array;  /* Array for GETALL, SETALL */  
    struct seminfo  *__buf;  /* Buffer for IPC_INFO (Linux-specific)*/  
};

// struct semid_ds : Linux 内核为 System V 信号量维护的数据结构  
struct semid_ds  
{  
    struct ipc_perm sem_perm;  /* Ownership and permissions */  
    time_t          sem_otime; /* Last semop time */  
    time_t          sem_ctime; /* Last change time */  
    unsigned long   sem_nsems; /* No. of semaphores in set */  
};  
```

示例，

```c++
/** 示例1: 将信号量集semid中的第一个信号量的值设置成为value(SETVAL) 
注意: semun联合体需要自己给出(从man-page中拷贝出来即可) 
**/  
union semun  
{  
    int              val;    /* Value for SETVAL */  
    struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET */  
    unsigned short  *array;  /* Array for GETALL, SETALL */  
    struct seminfo  *__buf;  /* Buffer for IPC_INFO 
                                           (Linux-specific) */  
};  
int sem_setval(int semid, int value)  
{  
    union semun su;  
    su.val = value;  
    if (semctl(semid, 0, SETVAL, su) == -1)  
        err_exit("sem_setval error");  
    return 0;  
}  

/** 示例2: 获取信号量集中第一个信号所关联的值(GETVAL) 
注意: 此时第四个参数可以不填, 而信号量所关联的值可以通过semctl的返回值返回(the value of semval.) 
**/  
int sem_getval(int semid)  
{  
    int value = semctl(semid, 0, GETVAL);  
    if (value == -1)  
        err_exit("sem_getval error");  
    return value;  
    return 0;  
}

/** 示例3: 删除一个信号量集(注意是删除整个集合) 
    IPC_RMID  Immediately  remove(立刻删除)  the  semaphore  set,  awakening  all  processes blocked in semop(2) calls on the set (with an error return and errno set to  EIDRM)[然后唤醒所有阻塞在该信号量上的进程]. The  argument  semnum  is ignored[忽略第二个参数]. 
**/  
int sem_delete(int semid)  
{  
    if (semctl(semid, 0, IPC_RMID) == -1)  
        err_exit("sem_delete error");  
    return 0;  
}  
  
//测试代码  
int main(int argc,char *argv[])  
{  
    int semid = sem_create(0x1234); //创建一个信号量集  
    sem_setval(semid, 500);         //设置值  
    cout << sem_getval(semid) << endl;  //获取值  
    sleep(10);  
    sem_delete(semid);      //删除该集合  
}

/**示例4: 获取/设置信号量的权限 
注意:一定要设定struct semid_ds结构体, 以指定使用semun的哪个字段 
**/  
int sem_getmode(int semid)  
{  
    union semun su;  
  
    // 注意: 下面这两行语句一定要设定.  
    // (告诉内核使用的semun的哪个字段)  
    struct semid_ds sd;  
    su.buf = &sd;  
    //  
    if (semctl(semid, 0, IPC_STAT, su) == -1)  
        err_exit("sem_getmode error");  
    printf("current permissions is: %o\n", su.buf->sem_perm.mode);  
    return 0;  
}  
int sem_setmode(int semid, char *mode)  
{  
    union semun su;  
    // 注意: 下面这两行语句一定要设定.  
    // (告诉内核使用的semun的哪个字段)  
    struct semid_ds sd;  
    su.buf = &sd;  
    //  
    sscanf(mode, "%o", (unsigned int *)&su.buf->sem_perm.mode);  
  
    if (semctl(semid, 0, IPC_SET, su) == -1)  
        err_exit("sem_setmode error");  
    return 0;  
}
```

**用来操纵一个信号量集, 以实现 P,V 操作**

```c
int semop(int semid, struct sembuf *sops, unsigned nsops);
```

semid: 是该信号量的标识码，也就是 semget 函数的返回值

sops: 是个指向一个结构数组(如果信号量集中只有一个信号量的话, 只有一个结构体也可)的指针

nsops: 所设置的信号量个数(如果 nsops>1 话, 需要将 sops 设置成为一个结构数组), 第三个参数表示第二个参数的对象的个数;

```c
//sembuf结构体  
struct sembuf  
{  
    unsigned short sem_num;  /* semaphore number:信号量的编号(从 0 开始) */  
    short          sem_op;   /* semaphore operation(+1, 0, -1) */  
    short          sem_flg;  /* operation flags: 常用取值为SEM_UNDO(解释见下) */  
};
```

sem_op 是信号量一次 PV 操作时加减的数值，一般只会用到两个值，一个是“-1”，也就是 P 操作，等待信号量变得可用；另一个是“+1”，也就是 V 操作，发出信号量已经变得可用, 该参数还可以等于 0, 表示进程将阻塞直到信号量的值等于 0;

sem_flg 有三个取值: SEM_UNDO(在进程结束时, 将该进程对信号量的操作复原，即取消该进程对信号量所有的操作, 推荐使用, IPC_NOWAIT(非阻塞)或 0 (默认操作, 并不撤销操作)。

```c++
/** 示例: P,V 操作封装  
**可以将 sembuf 的第三个参数设置为 IPC_NOWAIT/0, 以查看程序的状态的变化 
**/  
int sem_P(int semid)  
{  
    struct sembuf sops = {0, -1, SEM_UNDO};  
    if (semop(semid, &sops, 1) == -1)  
        err_exit("sem_P error");  
    return 0;  
}  
int sem_V(int semid)  
{  
    struct sembuf sops = {0, +1, SEM_UNDO};  
    if (semop(semid, &sops, 1) == -1)  
        err_exit("sem_V error");  
    return 0;  
}  
```

## 示例

```c++
//Usage.h  
#ifndef USAGE_H_INCLUDED  
#define USAGE_H_INCLUDED  
  
#include <iostream>  
#include <string>  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <string.h>  
  
#include <sys/types.h>  
#include <sys/stat.h>  
#include <sys/ipc.h>  
#include <sys/wait.h>  
#include <sys/time.h>  
#include <sys/msg.h>  
#include <sys/shm.h>  
#include <sys/mman.h>  
#include <sys/sem.h>  
#include <fcntl.h>  
#include <signal.h>  
#include <unistd.h>  
#include <grp.h>  
#include <pwd.h>  
#include <time.h>  
#include <errno.h>  
#include <mqueue.h>  
using namespace std;  
inline void err_quit(std::string message);  
inline void err_exit(std::string message);  
  
void usage()  
{  
    cerr << "Usage:" << endl;  
    cerr << "./semtool -c        #create" << endl;  
    cerr << "./semtool -d        #delte" << endl;  
    cerr << "./semtool -p        #signal" << endl;  
    cerr << "./semtool -v        #wait" << endl;  
    cerr << "./semtool -s <val>  #set-value" << endl;  
    cerr << "./semtool -g        #get-value" << endl;  
    cerr << "./semtool -f        #print-mode" << endl;  
    cerr << "./semtool -m <mode> #set-mode" << endl;  
}  
  
int sem_create(key_t key)  
{  
    int semid = semget(key, 1, IPC_CREAT|IPC_EXCL|0666);  
    if (semid == -1)  
        err_exit("sem_create error");  
    return semid;  
}  
int sem_open(key_t key)  
{  
    int semid = semget(key, 0, 0);  
    if (semid == -1)  
        err_exit("sem_open error");  
    return semid;  
}  
  
union semun  
{  
    int              val;    /* Value for SETVAL */  
    struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET */  
    unsigned short  *array;  /* Array for GETALL, SETALL */  
    struct seminfo  *__buf;  /* Buffer for IPC_INFO (Linux-specific) */  
};  
  
int sem_getmode(int semid)  
{  
    union semun su;  
  
    // 注意: 下面这两行语句一定要设定.  
    // (告诉内核使用的semun的哪个字段)  
    struct semid_ds sd;  
    su.buf = &sd;  
    //  
    if (semctl(semid, 0, IPC_STAT, su) == -1)  
        err_exit("sem_getmode error");  
    printf("current permissions is: %o\n", su.buf->sem_perm.mode);  
    return 0;  
}  
int sem_setmode(int semid, char *mode)  
{  
    union semun su;  
    // 注意: 下面这两行语句一定要设定.  
    // (告诉内核使用的semun的哪个字段)  
    struct semid_ds sd;  
    su.buf = &sd;  
    //  
    sscanf(mode, "%o", (unsigned int *)&su.buf->sem_perm.mode);  
  
    if (semctl(semid, 0, IPC_SET, su) == -1)  
        err_exit("sem_setmode error");  
    return 0;  
}  
int sem_getval(int semid)  
{  
    int value = semctl(semid, 0, GETVAL);  
    if (value == -1)  
        err_exit("sem_getval error");  
    cout << "current value: " << value << endl;  
    return value;  
}  
int sem_setval(int semid, int value)  
{  
    union semun su;  
    su.val = value;  
    if (semctl(semid, 0, SETVAL, su) == -1)  
        err_exit("sem_setval error");  
    return 0;  
}  
  
int sem_delete(int semid)  
{  
    if (semctl(semid, 0, IPC_RMID) == -1)  
        err_exit("sem_delete error");  
    return 0;  
}  
  
// 为了能够打印信号量的持续变化, 因此sem_flg我们并没用SEM_UNDO  
// 但是我们推荐使用SEM_UNDO  
int sem_P(int semid)  
{  
    struct sembuf sops = {0, -1, 0};  
    if (semop(semid, &sops, 1) == -1)  
        err_exit("sem_P error");  
    return 0;  
}  
int sem_V(int semid)  
{  
    struct sembuf sops = {0, +1, 0};  
    if (semop(semid, &sops, 1) == -1)  
        err_exit("sem_V error");  
    return 0;  
}  
  
inline void err_quit(std::string message)  
{  
    std::cerr << message << std::endl;  
    exit(EXIT_FAILURE);  
}  
inline void err_exit(std::string message)  
{  
    perror(message.c_str());  
    exit(EXIT_FAILURE);  
}  
  
#endif // USAGE_H_INCLUDED
```

```c++
/** 信号量综合运用示例: 
编译完成之后, 直接运行./semtool, 程序将打印该工具的用法; 
下面的这些函数调用, 只不过是对上面所封装函数的稍稍改动, 理解起来并不困难; 
**/  
//semtool.cpp  
#include "Usage.h"  
  
int main(int argc,char *argv[])  
{  
    int opt = getopt(argc, argv, "cdpvs:gfm:");  
    if (opt == '?')  
        exit(EXIT_FAILURE);  
    else if (opt == -1)  
    {  
        usage();  
        exit(EXIT_FAILURE);  
    }  
  
    key_t key = ftok(".", 's');  
    int semid;  
    switch (opt)  
    {  
    case 'c':  
        sem_create(key);  
        break;  
    case 'd':  
        semid = sem_open(key);  
        sem_delete(semid);  
        break;  
    case 'p':  
        semid = sem_open(key);  
        sem_P(semid);  
        sem_getval(semid);  
        break;  
    case 'v':  
        semid = sem_open(key);  
        sem_V(semid);  
        sem_getval(semid);  
        break;  
    case 's':  
        semid = sem_open(key);  
        sem_setval(semid, atoi(optarg));  
        sem_getval(semid);  
        break;  
    case 'g':  
        semid = sem_open(key);  
        sem_getval(semid);  
        break;  
    case 'f':  
        semid = sem_open(key);  
        sem_getmode(semid);  
        break;  
    case 'm':  
        semid = sem_open(key);  
        sem_setmode(semid, argv[2]);  
        sem_getmode(semid);  
        break;  
    default:  
        break;  
    }  
  
    return 0;  
}  
```
