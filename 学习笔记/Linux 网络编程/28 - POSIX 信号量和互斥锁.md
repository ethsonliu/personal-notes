## 目录

- [POSIX 信号量](#POSIX-信号量)
- [POSIX 互斥锁](#POSIX-互斥锁)
- [自旋锁和读写锁](#自旋锁和读写锁)

## POSIX 信号量

### 有名（named）信号量

```c
sem_t *sem_open(const char *name, int oflag); // 创建
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);

int sem_wait(sem_t *sem); // 如果信号量的值大于 0，减一并立即返回；如果等于 0，就阻塞直至大于 0
int sem_trywait(sem_t *sem); // 如果信号量的值等于 0，返回一个错误并且 errno 置为 EAGAIN；大于 0 时的行为和 sem_wait 行为一样
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout); // http://man7.org/linux/man-pages/man3/sem_timedwait.3.html
int sem_post(sem_t *sem); // 信号量的值加一
int sem_getvalue(sem_t *sem, int *sval); // （顾名思义）

int sem_close(sem_t *sem); // 
int sem_unlink(const char *name); // 
```

一个进程关闭了之后，内核会自动的对其上打开的的所有有名信号量自动执行关闭，但这并不代表就删除了此信号量。每个信号量都有一个应用计数器记录当前的打开
次数，当记录数大于 0 时，unlink 就将其从文件系统中删除，但是其信号的析构要等到最后一个 sem_close 发生时为止。

sem_unlink 会马上删除指定的信号量名，但要等到所有打开该信号量的进程关闭该信号量后才删除该信号。详细地说，当进程创建一个有名信号量，
会在 /dev/shm 下生成一个 sem.xxx 的文件，所有打开该信号量的进程（包括创建它的进程）都会增加该文件的引用计数，并且这个计数由内核管理。
当调用 sem_unlin k时，/dev/shm 下的 sem.xxx 文件会马上被删除，但是信号量本身并没有被删除，所有已打开该信号量的进程仍能正常使用它。直到所
有打开该信号量的进程关闭该信号量后，内核才真正删除信号量。

### 基于内存的（memory-based）信号量，也叫无名（unnamed）信号量

```c
int sem_init(sem_t *sem, int pshared, unsigned int value); // 创建

int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
int sem_post(sem_t *sem);
int sem_getvalue(sem_t *sem, int *sval);

int sem_destroy(sem_t *sem); // 销毁
```

## POSIX 互斥锁

```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex, const struct timespec *restrict abs_timeout);

int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

**创建互斥锁**

所以下面简单看一下如何创建和使用互斥锁。

在使用互斥锁之前，需要先创建一个互斥锁的对象。 互斥锁的类型是 pthread_mutex_t ，所以定义一个变量就是创建了一个互斥锁。

```c
pthread_mutex_t mtx;
```

这就定义了一个互斥锁。但是如果想使用这个互斥锁还是不行的，我们还需要对这个互斥锁进行初始化， 使用函数 pthread_mutex_init() 对互斥锁进行初始化操作。

```c
// 第二个参数是 NULL 的话，互斥锁的属性会设置为默认属性
pthread_mutex_init(&mtx, NULL);
```

除了使用 pthread_mutex_init() 初始化一个互斥锁，我们还可以使用下面的方式定义一个互斥锁：

```c
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
```

在头文件 /usr/include/pthread.h 中，对 PTHREAD_MUTEX_INITIALIZER 的声明如下

```c
# define PTHREAD_MUTEX_INITIALIZER \
   { { 0, 0, 0, 0, 0, 0, { 0, 0 } } }
```

为什么可以这样初始化呢，因为互斥锁的类型 pthread_mutex_t 是一个联合体， 其声明在文件 /usr/include/bits/pthreadtypes.h 中，代码如下：

```c
/* Data structures for mutex handling.  The structure of the attribute
   type is not exposed on purpose.  */
typedef union
{
    struct __pthread_mutex_s
    {
        int __lock;
        unsigned int __count;
        int __owner;
#if __WORDSIZE == 64
        unsigned int __nusers;
#endif
        /* KIND must stay at this position in the structure to maintain
           binary compatibility.  */
        int __kind;
#if __WORDSIZE == 64
        int __spins;
        __pthread_list_t __list;
# define __PTHREAD_MUTEX_HAVE_PREV  1
#else
        unsigned int __nusers;
        __extension__ union
        {
            int __spins;
            __pthread_slist_t __list;
        };
#endif
    } __data;
    char __size[__SIZEOF_PTHREAD_MUTEX_T];
    long int __align;
} pthread_mutex_t;
```
 

**获取互斥锁**

接下来是如何使用互斥锁进行互斥操作。在进行互斥操作的时候， 应该先"拿到锁"再执行需要互斥的操作，否则可能会导致多个线程都需要访问的数据结果不一致。 例如在一个线程在试图修改一个变量的时候，另一个线程也试图去修改这个变量， 那就很可能让后修改的这个线程把前面线程所做的修改覆盖了。

下面是获取锁的操作：

**阻塞调用**

```c
pthread_mutex_lock(&mtx);
```

这个操作是阻塞调用的，也就是说，如果这个锁此时正在被其它线程占用， 那么 pthread_mutex_lock() 调用会进入到这个锁的排队队列中，并会进入阻塞状态， 直到拿到锁之后才会返回。

**非阻塞调用**

如果不想阻塞，而是想尝试获取一下，如果锁被占用咱就不用，如果没被占用那就用， 这该怎么实现呢？可以使用 pthread_mutex_trylock() 函数。 这个函数和 pthread_mutex_lock() 用法一样，只不过当请求的锁正在被占用的时候， 不会进入阻塞状态，而是立刻返回，并返回一个错误代码 EBUSY，意思是说， 有其它线程正在使用这个锁。

```c
int err = pthread_mutex_trylock(&mtx);
if(0 != err) {
    if(EBUSY == err) {
        //The mutex could not be acquired because it was already locked.
    }
}
```
 

**超时调用**

如果不想不断的调用 pthread_mutex_trylock() 来测试互斥锁是否可用， 而是想阻塞调用，但是增加一个超时时间呢，那么可以使用 pthread_mutex_timedlock() 来解决， 其调用方式如下：

```c
struct timespec abs_timeout;
abs_timeout.tv_sec = time(NULL) + 1;
abs_timeout.tv_nsec = 0;

int err = pthread_mutex_timedlock(&mtx, &abs_timeout);
if(0 != err) {
    if(ETIMEDOUT == err) {
        //The mutex could not be locked before the specified timeout expired.
    }
}
```

上面代码的意思是，阻塞等待线程锁，但是只等1秒钟，一秒钟后如果还没拿到锁的话， 那就返回，并返回一个错误代码 ETIMEDOUT，意思是超时了。

其中 timespec 定义在头文件 time.h 中，其定义如下

```c
struct timespec
{
    __time_t tv_sec;        /* Seconds.  */
    long int tv_nsec;       /* Nanoseconds.  */
};
```

还需要注意的是，这个函数里面的时间，是绝对时间，所以这里用 time() 函数返回的时间增加了 1 秒。

**释放互斥锁**

用完了互斥锁，一定要记得释放，不然下一个想要获得这个锁的线程， 就只能去等着了，如果那个线程很不幸的使用了阻塞等待，那就悲催了。

释放互斥锁比较简单，使用 pthread_mutex_unlock() 即可：

```
pthread_mutex_unlock(&mtx);
```

**销毁线程锁**

通过 man pthread_mutex_destroy 命令可以看到 pthread_mutex_destroy() 函数的说明， 在使用此函数销毁一个线程锁后，线程锁的状态变为"未定义"。有的 pthread_mutex_destroy 实现方式，会使线程锁变为一个不可用的值。一个被销毁的线程锁可以被 pthread_mutex_init() 再次初始化。对被销毁的线程锁进行其它操作，其结果是未定义的。

对一个处于已初始化但未锁定状态的线程锁进行销毁是安全的。尽量避免对一个处于锁定状态的线程锁进行销毁操作。

销毁线程锁的操作如下：

```c
pthread_mutex_destroy(&mtx)
```

## 自旋锁和读写锁








































