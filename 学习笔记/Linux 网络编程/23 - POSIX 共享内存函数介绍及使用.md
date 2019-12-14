## 目录

- [函数介绍](#函数介绍)
- [](#)

## 函数介绍

**创建或打开一个共享内存对象**

```c
#include <sys/mman.h>
#include <sys/stat.h>        /* For mode constants */
#include <fcntl.h>           /* For O_* constants */

int shm_open(const char *name, int oflag, mode_t mode);
int shm_unlink(const char *name);
```

name：共享内存对象的名字

oflag：与 open 函数类似

mode：和 open 函数的 mode 一样，是指定权限位。如果没有指定 O_CREAT 标志，那么该参数可以指定为 0

返回：若成功则非负描述符；若出错则为 -1。

**修改共享内存大小**

```c
#include <unistd.h>
#include <sys/types.h>

int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

fd：文件描述符

length：长度

返回值：成功返回 0；失败返回 -1

**删除一个共享内存对象**

```c
#include <sys/mman.h>
#include <sys/stat.h> /* For mode constants */
#include <fcntl.h> /* For O_* constants */

int shm_unlink(const char *name);
```

name：共享内存对象的名字

返回值：成功返回 0；失败返回 -1。










































