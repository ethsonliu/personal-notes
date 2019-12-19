## 目录

- [pthread_create](#pthread_create)
- [pthread_exit](#pthread_exit)
- [pthread_join](#pthread_join)
- [pthread_create](#pthread_create)

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

retval: 它指向一个指针，后者指向线程的返回值

返回值：成功返回 0；失败返回错误码









