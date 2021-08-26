基本概念：

Linux 内核有个机制叫 OOM killer(Out Of Memory killer)，该机制会监控那些占用内存过大，尤其是瞬间占用内存很快的进程，然后防止内存耗尽而自动把该进程杀掉。
内核检测到系统内存不足、挑选并杀掉某个进程的过程可以参考内核源代码 linux/mm/oom_kill.c，当系统内存不足的时候，out_of_memory() 被触发，然后调用 select_bad_process() 
选择一个 bad 进程杀掉。如何判断和选择一个 bad 进程呢？linux 选择 bad 进程是通过调用 oom_badness()，挑选的算法和想法都很简单很朴实：最 bad 的那个进程就是那个最占用内存的进程。

如何查看：

```
grep "Out of memory" /var/log/messages`
```

查看系统日志方法：

运行 `egrep -i -r 'killed process' /var/log` 命令，也可直接运行 dmesg 命令。

