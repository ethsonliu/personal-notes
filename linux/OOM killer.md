**基本概念**

Linux 内核有个机制叫 OOM killer(Out Of Memory killer)，该机制会监控那些占用内存过大，尤其是瞬间占用内存很快的进程，然后防止内存耗尽而自动把该进程杀掉。
内核检测到系统内存不足、挑选并杀掉某个进程的过程可以参考内核源代码 linux/mm/oom_kill.c，当系统内存不足的时候，out_of_memory() 被触发，然后调用 select_bad_process() 
选择一个 bad 进程杀掉。如何判断和选择一个 bad 进程呢？linux 选择 bad 进程是通过调用 oom_badness()，挑选的算法和想法都很简单很朴实：最 bad 的那个进程就是那个最占用内存的进程。

**如何查看**

```
grep "Out of memory" /var/log/messages`
```

查看系统日志方法，

运行 `egrep -i -r 'killed process' /var/log` 命令，

```
root@tms-91:~# egrep -i -r 'killed process' /var/log
/var/log/kern.log:Aug 25 14:06:20 tms-91 kernel: Killed process 1149 (pengine) total-vm:6529372kB, anon-rss:2237192kB, file-rss:12420kB
/var/log/kern.log:Aug 25 14:11:51 tms-91 kernel: Killed process 31115 (tcm) total-vm:8891124kB, anon-rss:3644268kB, file-rss:0kB
/var/log/kern.log:Aug 25 14:36:02 tms-91 kernel: Killed process 5696 (tcm) total-vm:8953588kB, anon-rss:3684952kB, file-rss:0kB
/var/log/kern.log:Aug 25 15:14:01 tms-91 kernel: Killed process 6332 (tcm) total-vm:8810228kB, anon-rss:3740052kB, file-rss:0kB
/var/log/kern.log:Aug 25 15:40:51 tms-91 kernel: Killed process 6632 (tcm) total-vm:8810228kB, anon-rss:3761188kB, file-rss:0kB
/var/log/kern.log:Aug 25 16:01:07 tms-91 kernel: Killed process 6820 (tcm) total-vm:8875764kB, anon-rss:3745340kB, file-rss:0kB
/var/log/kern.log:Aug 26 10:18:27 tms-91 kernel: Killed process 16957 (tcm) total-vm:8747764kB, anon-rss:3723168kB, file-rss:0kB
/var/log/error:Sep 25 10:49:55 tms-91 kernel: Killed process 1120 (pengine) total-vm:9715232kB, anon-rss:3626388kB, file-rss:4232kB
/var/log/error:Mar 18 16:27:09 tms-91 kernel: Killed process 1070 (pengine) total-vm:131519040kB, anon-rss:3516464kB, file-rss:9740kB
/var/log/error:Mar 25 13:08:43 tms-91 kernel: Killed process 22267 (pengine) total-vm:180953540kB, anon-rss:3446276kB, file-rss:13540kB
/var/log/error:Aug 25 14:06:20 tms-91 kernel: Killed process 1149 (pengine) total-vm:6529372kB, anon-rss:2237192kB, file-rss:12420kB
/var/log/error:Aug 25 14:11:51 tms-91 kernel: Killed process 31115 (tcm) total-vm:8891124kB, anon-rss:3644268kB, file-rss:0kB
/var/log/error:Aug 25 14:36:02 tms-91 kernel: Killed process 5696 (tcm) total-vm:8953588kB, anon-rss:3684952kB, file-rss:0kB
/var/log/error:Aug 25 15:14:01 tms-91 kernel: Killed process 6332 (tcm) total-vm:8810228kB, anon-rss:3740052kB, file-rss:0kB
/var/log/error:Aug 25 15:40:51 tms-91 kernel: Killed process 6632 (tcm) total-vm:8810228kB, anon-rss:3761188kB, file-rss:0kB
/var/log/error:Aug 25 16:01:07 tms-91 kernel: Killed process 6820 (tcm) total-vm:8875764kB, anon-rss:3745340kB, file-rss:0kB
/var/log/error:Aug 26 10:18:27 tms-91 kernel: Killed process 16957 (tcm) total-vm:8747764kB, anon-rss:3723168kB, file-rss:0kB
```

也可直接运行 dmesg 命令。

