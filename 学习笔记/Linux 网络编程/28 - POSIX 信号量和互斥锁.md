## 目录

- [POSIX 信号量](#POSIX-信号量)
- [POSIX 互斥锁](#POSIX-互斥锁)
- [生产者消费者](#生产者消费者)
- [自旋锁和读写锁](#自旋锁和读写锁)

## POSIX 信号量

### 有名（named）信号量

```
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

```
int sem_init(sem_t *sem, int pshared, unsigned int value);

int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
int sem_post(sem_t *sem);
int sem_getvalue(sem_t *sem, int *sval);

int sem_destroy(sem_t *sem);
```

## POSIX 互斥锁


## 生产者消费者


## 自旋锁和读写锁








































