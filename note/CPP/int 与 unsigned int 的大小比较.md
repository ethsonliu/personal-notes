先看一段很简单的 Linux 代码，

```c++
int n = read(fd, buffer, sizeof(buffer));

if (n < sizeof(buffer))
    exit(EXIT_FAILURE);
else
    do_something();
```
这段代码是不安全的。

假如`read`返回 -1，按照预期，代码应该执行`exit(EXIT_FAILURE)`退出程序，但实际真的是这样么？

一个 unsigned int 类型的数与 int 类型的数进行比较运算，int 类型的数会被自动转换为 unsigned int 类型。也就是说 int 类型的 -1 在进行比较时会被转换成一个非常大的 unsigned int 类型的数（具体数值大小是`(1 << 32) - 1`）。