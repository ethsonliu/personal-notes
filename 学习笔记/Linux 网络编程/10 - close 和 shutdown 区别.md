```c
#include <unistd.h>
int close(int fd);
```

close 只会减少一次引用次数，如果引用次数为 0，则关闭自身数据传输的两个方向。主要是在 fork 的调用中出现，因为文件描述符会被复制到另一个进程中。

```c
#include <sys/socket.h>
int shutdown(int sockfd, int how);
```

shutdown 可以选择关闭某个方向或者同时关闭两个方向，shutdown how = 0 or how = 1 or how = 2 (SHUT_RD or SHUT_WR or SHUT_RDWR)。
