## 目录

- [信号量简介](#信号量简介)
- [相关函数](#相关函数)
- [示例](#示例)

## 信号量简介

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行进程访问代码的临
界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个进程在访问它，也就是说信
号量是用来调协进程对共享资源的访问的。

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





## 示例
