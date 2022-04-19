journal 和 WAL 是 sqlite 解决原子事务和安全 rollback 的两种手段，前者是以前的版本使用的，后因并发性能不足，sqlite 推出了后者。

## journal

在做嵌入式平台开发时使用 sqlite 会发现数据库的旁边生成了一个大小为 0 的与数据库文件同名的.db-journal 文件。该文件是 sqlite 的一个临时的日志文件，主要用于 sqlite 事务回滚
机制，在事务开始时产生，在事务结束时删除；当程序发生崩溃或者系统断电时该文件将留在磁盘上，以便下次程序运行时进行事务回滚。

但是我创建数据库时将事务结束了，同时程序也没有崩溃，为什么还是会在磁盘上产生 .db-journal 文件呢？

深入研究，发现这是 sqlite 生成日志文件的不同模式造成的，在 android 采用的这种模式下，.db-journal 文件是永久的留在磁盘上不会被自动清除的，如果没有发生事务回滚那么 .db-journal 
文件的大小为 0，这样就避免了每次生成和删除 .db-journal 文件的开销。

关于 journal_mode 的介绍（https://www.sqlite.org/pragma.html#pragma_journal_mode），

```
PRAGMA schema.journal_mode;
PRAGMA schema.journal_mode = DELETE | TRUNCATE | PERSIST | MEMORY | WAL | OFF
```

- DELETE，这是默认值，事务开始 journal 文件就会创建，事务结束文件就会删掉。
- TRUNCATE，相比 DELETE，不是删除文件，只是把文件内容清空。
- PERSIST，事务结束后文件不会被删除，但是文件头部会用 0 填充，这样其它的 connections 就无法使用该文件恢复数据库。这种模式适合 DELETE 和 TRUNCATE 的代价比 PERSIST 大的系统平台上。
- MEMORY，文件直接存储在内存中而不是磁盘上，但这种模式也比较危险。如果程序崩溃的时候事务刚写一半，可能数据库文件都会损坏。
- OFF，禁用 rollback journal 功能，这会使 sqlite 的原子事务和安全 rollback 全部失效，最终的 sql 语句执行行为将无法明确，程序崩溃或断电也可能造成数据库文件损坏。
- WAL，见下面的讲解。

Note also that the journal_mode cannot be changed while a transaction is active.

## WAL（Write-Ahead Logging）

这是 version 3.7.0 (2010-07-21) 引入的新机制。

之前的 journal 做法是，在修改数据库文件数据之前，先将修改所在分页中的数据备份在 journal 文件中，然后再将修改写入到数据库文件中；如果事务失败，则将备份数据拷贝回来，撤销修改；如果事务成功，则删除备份数据，提交修改。

WAL 机制的原理是：修改并不直接写入到数据库文件中，而是写入到另外一个称为 WAL 的文件中；如果事务失败，WAL 中的记录会被忽略，撤销修改；如果事务成功，它将在随后的某个时间被写回到数据库文件中，提交修改。

同步 WAL 文件和数据库文件的行为被称为 checkpoint（检查点），它由 SQLite 自动执行，默认是在 WAL 文件积累到 1000 页修改的时候；当然，在适当的时候，也可以手动执行 checkpoint，SQLite 提供了相关的接口。执行 checkpoint 之后，WAL 文件会被清空。

在读的时候，SQLite 将在 WAL 文件中搜索，找到最后一个写入点，记住它，并忽略在此之后的写入点（这保证了读写和读读可以并行执行）；随后，它确定所要读的数据所在页是否在 WAL 文件中，如果在，则读 WAL 文件中的数据，如果不在，则直接读数据库文件中的数据。

在写的时候，SQLite 将之写入到 WAL 文件中即可，但是必须保证独占写入，因此写写之间不能并行执行。

WAL 在实现的过程中，也会使用共享内存技术（会创建一个 -shm 文件），因此，所有的读写进程必须在同一个机器上，否则，无法保证数据一致性。这个文件时为了提高性能，里边是一个内存索引(-shm)，映射每一个 page 是否 dirty，读取时先看需要的 page 是否在 WAL 日志中，然后再读取。

Page 是 SQLite 的基本数据存储单元，类似于文件系统的 block，SQLite 中 Page 大小为 [512, 65536]Byte。

## 参考

- https://zcw.me/blogwp/sqlite-wal%E6%9C%BA%E5%88%B6%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/
- https://www.cnblogs.com/frydsh/archive/2013/04/13/3018666.html
