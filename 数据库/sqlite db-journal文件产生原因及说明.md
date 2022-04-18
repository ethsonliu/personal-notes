今天在 Android 中将 sqlite 的数据库文件生成在SD卡上的过程中，发现生成的.db文件的旁边

在做嵌入式平台开发时使用 sqlite 会发现数据库的旁边生成了一个大小为 0 的与数据库文件同名的.db-journal 文件。该文件是 sqlite 的一个临时的日志文件，主要用于 sqlite 事务回滚
机制，在事务开始时产生，在事务结束时删除；当程序发生崩溃或者系统断电时该文件将留在磁盘上，以便下次程序运行时进行事务回滚。

但是我创建数据库时将事务结束了，同时程序也没有崩溃，为什么还是会在磁盘上产生 .db-journal 文件呢？

深入研究，发现这是 sqlite 生成日志文件的不同模式造成的，在 android 采用的这种模式下，.db-journal 文件是永久的留在磁盘上不会被自动清除的，如果没有发生事务回滚那么 .db-journal 
文件的大小为 0，这样就避免了每次生成和删除 .db-journal 文件的开销。

关于 journal_mode 的介绍，

- https://stackoverflow.com/questions/633274/what-causes-a-journal-file-to-be-created-in-sqlite
- https://www.runoob.com/sqlite/sqlite-pragma.html
