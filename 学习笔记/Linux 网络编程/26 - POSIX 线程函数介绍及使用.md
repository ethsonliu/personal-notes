## 目录

- [pthread_create](#pthread_create)
- [pthread_exit](#pthread_exit)
- [pthread_join](#pthread_join)
- [pthread_self](#pthread_self)
- [pthread_cancel](#pthread_cancel)
- [pthread_detach](#pthread_detach)

## pthread_create

创建一个新的线程并运行。

```c
#include <pthread.h>

int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
```

thread: 返回的线程 ID

attr: 设置线程的属性，attr 为 NULL 表示使用默认属性

start_routine: 是个函数地址，线程启动后要执行的函数

arg: 传给线程启动函数的参数

返回值：成功返回 0；失败返回错误码。

```c++
#include <pthread.h>
#include <stdio.h>

void* thread_test(void* ptr)
{
    while(1)
        printf("i am pthread\n");
}

int main()
{
    pthread_t pid;
    pthread_create(&pid, NULL, test_thread, NULL);
    
    while(1)
        printf("i am main pthread\n");
        
    return 0;
}
```

## pthread_exit

终止线程。

```c
#include <pthread.h>

void pthread_exit(void *retval);
```

retval：返回 pthread_join 的第二个参数的值。

返回值：无返回值，跟进程一样，线程结束的时候无法返回到它的调用者（自身）

如果需要只终止某个线程而不终止整个进程，可以有三种方法：

1. 从线程函数 return。这种方法对主线程不适用，从 main 函数 return 相当于调用 exit，而如果任意一个线程调用了 exit 或 _exit，则整个进程的所有线程都
终止。

2. 一个线程可以调用 pthread_cancel 终止同一进程中的另一个线程。

3. 线程可以调用 pthread_exit 终止自己。

## pthread_join

执行线程并阻塞等待线程结束。

```c
#include <pthread.h>

int pthread_join(pthread_t thread, void **retval);
```

thread: 线程 ID

retval: 用户定义的指针，用来存储被等待线程的返回值。

返回值：成功返回 0；失败返回错误码

## pthread_self

返回当前线程 ID。

```c
#include <pthread.h>

pthread_t pthread_self(void);
```

返回值：成功返回线程 id

在 Linux 上，pthread_t 类型是一个地址值，属于同一进程的多个线程调用 getpid(2)可以得到相同的进程号，而调用 pthread_self 得到的线程号各不相同。线程 id 只在当前进程中保证是唯一的，在不同的系统中 pthread_t 这个类型有不同的实现，它可能是一个整数值，也可能是一个结构体，也可能是一个地址，所以不能简单地当成整数用 printf 打印。

## pthread_cancel

取消一个执行中的线程。

```c
#include <pthread.h>

int pthread_cancel(pthread_t thread);
```

thread: 线程 ID

返回值：成功返回 0；失败返回错误码

一个新创建的线程默认取消状态（cancelability state）是可取消的，取消类型（ cancelability type）是同步的，即在某个可取消点（ cancellation point，即在执行某些函数的时候）才会取消线程。具体可以 man 一下。

相关函数 `int pthread_setcancelstate(int state, int *oldstate)` `int pthread_setcanceltype(int type, int *oldtype)` 为保证一个事务型处理逻辑的完整可以使用这两个函数，如下举例，主线程创建完线程睡眠一阵调用 pthread_cancel，test 是 thread_function。

```c++
	
void *test(void *arg)
{
    for (int i = 0; i < 10; i++)
    {
        pthread_setcancelstate(PTHREAD_CANCEL_DISABLE, NULL);
        printf("start: %d; ", i);
        sleep(1);
        printf("end: %d\n", i);
        if (i > 7)
        {
            pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);
            pthread_testcancel();
        }
    }
    return (void *)0;
}
```

## pthread_detach

将一个线程分离。

```c
#include <pthread.h>

int pthread_detach(pthread_t thread);
```

thread: 线程 ID

返回值：成功返回 0；失败返回错误码

一般情况下，线程终止后，其终止状态一直保留到其它线程调用 pthread_join 获取它的状态为止（僵线程）。但是线程也可以被置为 detach 状态，这样的线程一旦终止就立刻回收它占用的所有资源，而不保留终止状态。不能对一个已经处于 detach 状态的线程调用 pthread_join，这样的调用将返回 EINVAL。对一个尚未 detach 的线程调用 pthread_join 或 pthread_detach 都可以把该线程置为 detach 状态，也就是说，不能对同一线程调用两次 pthread_join，或者如果已经对一个线程调用了 pthread_detach 就不能再调用 pthread_join 了。

```c++
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

pthread_t tid;
pthread_create(&tid, &attr, test, "a"); // test is thread_function

sleep(3);

pthread_attr_destroy(&attr);
```
