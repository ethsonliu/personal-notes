## 目录

- [条件变量和互斥量的区别](#条件变量和互斥量的区别)
- [条件变量接口](#条件变量接口)
- [条件变量和互斥量搭配的使用模板](#条件变量和互斥量搭配的使用模板)
- [pthread_cond_wait](#pthread_cond_wait)
- [条件变量和互斥量实现生产者消费者](#条件变量和互斥量实现生产者消费者)


## 条件变量和互斥量的区别

- [https://stackoverflow.com/questions/4742196/advantages-of-using-condition-variables-over-mutex](https://stackoverflow.com/questions/4742196/advantages-of-using-condition-variables-over-mutex)


## 条件变量接口

```c
#include <pthread.h>

int pthread_cond_destroy(pthread_cond_t *cond);
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// unblock on a condition variable
int pthread_cond_broadcast(pthread_cond_t *cond); // for all
int pthread_cond_signal(pthread_cond_t *cond); // for at least one

// block on a condition variable
int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
```

## 条件变量和互斥量搭配的使用模板

第一，等待条件代码

```
pthread_mutex_lock(&mutex);
while (条件为假）
    pthread_cond_wait(&cond, &mutex);
修改条件;
pthread_mutex_unlock(&mutex);
```

第二，修改条件代码
```
pthread_mutex_lock(&mutex);
设置条件为真
pthread_cond_signal(&cond);
pthread_mutex_unlock(&mutex);
```

在这里的一个难点是，第一段条件等待代码为什么不用 if 而用 while 呢？在 man 3 pthread_cond_wait 之后发现这样一段话

If a signal is delivered to a thread waiting for a condition variable, upon return from the signal handler the thread resumes waiting for the condition variable as if it was not interrupted, or it shall return zero due to spurious wakeup.

也就是说，唤醒条件的信号，可以唤醒多个线程，但是只能允许一个信号访问，也就是说，等待线程需要不断的用 while 轮询一直到达到条件了才行。

假设有两个线程（我就用伪代码了）：

```
//thread 1
while(0<x<10)
    pthread_cond_wait(); // I

//thread 2
while(5<x<15)
    pthread_cond_wait(); // II
```

如果某段时间内 x == 8，那么两个线程相继进入等待。thread 1 停留在 I 处等待 phtread_cond_signal 到来 thread 2 停留在 II 处等待 phtread_cond_signal 到来。

然后，另一个线程 3 修改 x = 12，然后执行了 phtread_cond_signal。

如果 while 都换成 if 的话，那么线程 1、2 都被唤醒了，但是，此时 x==12，应该线程 1 继续等待才对，换成 while 的话就可避免了。


## pthread_cond_wait

关于 pthread_cond_wait 的具体实现，

参考：

- <https://www.zhihu.com/question/24116967>
- <https://android.googlesource.com/platform/external/pthreads/+/216eb8153b2455d1e15f8fbf84d4413ddcc40adc/pthread_cond_wait.c>

## 条件变量和互斥量实现生产者消费者

```c++
#include <unistd.h>
#include <sys/types.h>
#include <pthread.h>
#include <semaphore.h>

#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>

#define ERR_EXIT(m) \
        do \
        { \
                perror(m); \
                exit(EXIT_FAILURE); \
        } while(0)

#define CONSUMERS_COUNT 2
#define PRODUCERS_COUNT 1

pthread_mutex_t g_mutex;
pthread_cond_t g_cond;

pthread_t g_thread[CONSUMERS_COUNT + PRODUCERS_COUNT];

int nready = 0;

void *consume(void *arg)
{
    int num = (int)arg;
    while (1)
    {
        pthread_mutex_lock(&g_mutex);
        while (nready == 0)
        {
            printf("%d begin wait a condtion ...\n", num);
            pthread_cond_wait(&g_cond, &g_mutex);
        }

        printf("%d end wait a condtion ...\n", num);
        printf("%d begin consume product ...\n", num);
        --nready;
        printf("%d end consume product ...\n", num);
        pthread_mutex_unlock(&g_mutex);
        sleep(1);
    }
    return NULL;
}

void *produce(void *arg)
{
    int num = (int)arg;
    while (1)
    {
        pthread_mutex_lock(&g_mutex);
        printf("%d begin produce product ...\n", num);
        ++nready;
        printf("%d end produce product ...\n", num);
        pthread_cond_signal(&g_cond);
        printf("%d signal ...\n", num);
        pthread_mutex_unlock(&g_mutex);
        sleep(1);
    }
    return NULL;
}

int main(void)
{
    int i;

    pthread_mutex_init(&g_mutex, NULL);
    pthread_cond_init(&g_cond, NULL);


    for (i = 0; i < CONSUMERS_COUNT; i++)
        pthread_create(&g_thread[i], NULL, consume, (void *)i);

    sleep(1);

    for (i = 0; i < PRODUCERS_COUNT; i++)
        pthread_create(&g_thread[CONSUMERS_COUNT + i], NULL, produce, (void *)i);

    for (i = 0; i < CONSUMERS_COUNT + PRODUCERS_COUNT; i++)
        pthread_join(g_thread[i], NULL);

    pthread_mutex_destroy(&g_mutex);
    pthread_cond_destroy(&g_cond);

    return 0;
}
```

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/029.png)
