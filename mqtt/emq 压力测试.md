首先修改 emq 配置文件，修改下面的字段为这样：

```
 // 修改队列无限大
zone.external.max_mqueue_len = 0
zone.external.max_mqueue_len = 0
```

emq 在压力测试的时候，内存肯定会飙高，为了避免被 OOM Killer干掉，需要设置一下，参考文章 [Linux 如何保护重要进程不被OOM Killer干掉](https://blog.csdn.net/wtopps/article/details/89052550)。



## 参考

- [Linux 如何保护重要进程不被OOM Killer干掉](https://blog.csdn.net/wtopps/article/details/89052550)
- [一个进程能运行多少线程](https://www.cnblogs.com/wozijisun/p/10370897.html)
- [讨论：一个进程(Process)最多可以生成多少个线程(Thread)](https://blog.csdn.net/great3779/article/details/5930190)
