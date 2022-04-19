```
PRAGMA synchronous = FULL;
PRAGMA synchronous = NORMAL;
PRAGMA synchronous = OFF;
```

参数含义：

- FULL, SQLite 数据库引擎在紧急时刻会暂停以确定数据已经写入磁盘。这使 系统崩溃或电源出问题时能确保数据库在重起后不会损坏。FULL synchronous 很安全但很慢。
- NORMAL, SQLite 数据库引擎在大部分紧急时刻会暂停，但不像 FULL 模式下那么频繁。 NORMAL 模式下有很小的几率(但不是不存在)发生电源故障导致数据库损坏的情况。但实际上，在这种情况下很可能你的硬盘已经不能使用，或者发生了其他的不可恢复的硬件错误。
- OFF，SQLite 在传递数据给系统以后直接继续而不暂停。若运行 SQLite 的应用程序崩溃， 数据不会损伤，但在系统崩溃或写入数据时意外断电的情况下数据库可能会损坏。另一方面，在 synchronous OFF 时 一些操作可能会快 50 倍甚至更多。在 SQLite 2 中，缺省值为 NORMAL.而在 3 中修改为 FULL。

关于【SQLite 数据库引擎在紧急时刻会暂停以确定数据已经写入磁盘】这句话阐述的并不是很清楚，其如何实现的也根本没提，从网上找到的信息得到以下结论，

数据库为了执行备份，备份文件依赖于写磁盘，然后才会写到数据库文件磁盘中。关键在于这里的写磁盘，都是调用系统的 write 接口，绝大部分都是直接写缓冲区的，只有调用 sync 才会将缓冲区中的
数据 flush 到磁盘。所以在 write，sync，再 wirte 再 sync 的过程中，掉电后是否能恢复数据，依赖于 sync 是否有真正执行。从这个角度看，FULL 和 NORMAL 的区别，似乎就只有 sync 调用的频率，
FULL 按照多人的意见是一个 transaction 一个 sync，而 NORMAL 是多个 transactions 调一个 sync。

参考：https://blog.csdn.net/chinaclock/article/details/48622243
