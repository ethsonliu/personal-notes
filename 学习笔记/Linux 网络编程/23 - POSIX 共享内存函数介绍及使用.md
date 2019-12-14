## 目录

- [函数介绍](#函数介绍)
- [示例](#示例)

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

**示例**

```c++
#include <errno.h>
#include <unistd.h>
#include <iostream>
#include <stdio.h>
#include <cstring>

#include <fcntl.h>           /* For O_* constants */
#include <sys/stat.h>        /* For mode constants */
#include <mqueue.h>
#include <sys/mman.h>

using namespace std;

#define ERR_EXIT(m) \
        do  \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while (0);

struct student
{
    char name[32];
    int age;
};

int main(int argc, char** argv)
{
    int shmid;
    shmid = shm_open("/xyz", O_CREAT | O_RDWR, 0666);
    //shmid = shm_open("/xyz", O_RDWR, S_IRUSR);
    if (shmid == -1)
    {
        ERR_EXIT("shmid");
    }
    printf("shmopen success\n");

    student stu;
    strcpy(stu.name , "hello");
    stu.age = 20;

    if (ftruncate(shmid, sizeof(stu)) == -1)
    {
        ERR_EXIT("ftruncate");
    }
    printf("truncate succ\n");

    struct stat buf;
    if (fstat(shmid, &buf) == -1)
    {
        ERR_EXIT("fstat");
    }

    printf("mode: %o, size: %ld\n", buf.st_mode & 07777, buf.st_size);

    close(shmid);
    if (shm_unlink("/xyz") == -1)
    {
        ERR_EXIT("shmunlink");
    }
    printf("shm_unlink succ\n");


    return 0;
}
```

## 示例

```c++
// shmwrite.cpp

#include <errno.h>
#include <unistd.h>
#include <iostream>
#include <stdio.h>
#include <cstring>

#include <fcntl.h>           /* For O_* constants */
#include <sys/stat.h>        /* For mode constants */
#include <mqueue.h>
#include <sys/mman.h>

using namespace std;

#define ERR_EXIT(m) \
        do  \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while (0);

struct student
{
    char name[32];
    int age;
};

int main(int argc, char** argv)
{
    int shmid;
    shmid = shm_open("/xyz", O_CREAT | O_RDWR, 0666);
    //shmid = shm_open("/xyz", O_RDWR, S_IRUSR);
    if (shmid == -1)
    {
        ERR_EXIT("shmid");
    }
    printf("shmopen success\n");

    student* stu;


    if (ftruncate(shmid, sizeof(stu)) == -1)
    {
        ERR_EXIT("ftruncate");
    }
    printf("truncate succ\n");

    struct stat buf;
    if (fstat(shmid, &buf) == -1)
    {
        ERR_EXIT("fstat");
    }

    printf("mode: %o, size: %ld\n", buf.st_mode & 07777, buf.st_size);

    stu = (student*)mmap(NULL, buf.st_size, PROT_WRITE, MAP_SHARED, shmid, 0);

    strcpy(stu->name , "hello");
    stu->age = 20;

    close(shmid);

    return 0;
}
```

```c++
// shmreceive.cpp

#include <errno.h>
#include <unistd.h>
#include <iostream>
#include <stdio.h>
#include <cstring>

#include <fcntl.h>           /* For O_* constants */
#include <sys/stat.h>        /* For mode constants */
#include <mqueue.h>
#include <sys/mman.h>

using namespace std;

#define ERR_EXIT(m) \
        do  \
        {   \
            perror(m);  \
            exit(EXIT_FAILURE); \
        } while (0);

struct student
{
    char name[32];
    int age;
};

int main(int argc, char** argv)
{
    int shmid;
    shmid = shm_open("/xyz", O_CREAT | O_RDWR, 0666);
    //shmid = shm_open("/xyz", O_RDWR, S_IRUSR);
    if (shmid == -1)
    {
        ERR_EXIT("shmid");
    }
    printf("shmopen success\n");

    student* stu;
    if (ftruncate(shmid, sizeof(stu)) == -1)
    {
        ERR_EXIT("ftruncate");
    }
    printf("truncate succ\n");

    struct stat buf;
    if (fstat(shmid, &buf) == -1)
    {
        ERR_EXIT("fstat");
    }

    printf("mode: %o, size: %ld\n", buf.st_mode & 07777, buf.st_size);

    stu = (student*)mmap(NULL, buf.st_size, PROT_READ, MAP_SHARED, shmid, 0);
    printf("name: %s, age: %d\n", stu->name, stu->age);

    close(shmid);

    return 0;
}
```
