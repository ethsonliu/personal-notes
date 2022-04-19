**多线程**

https://www.sqlite.org/faq.html#q6

**多进程**

https://www.sqlite.org/faq.html#q5

**使用建议**

由此可见，要想保证线程安全的话，可以有这 4 种方式（https://www.sqlite.org/threadsafe.html）：

- SQLite 使用单线程模式，用一个专门的线程访问数据库。
- SQLite 使用单线程模式，用一个线程队列来访问数据库，队列一次只允许一个线程执行，队列里的线程共用一个数据库连接。
- SQLite 使用多线程模式，每个线程创建自己的数据库连接。
- SQLite 使用串行模式，所有线程共用全局的数据库连接。

**如果必须多进程访问，怎么解决 SQLITE_BUSY 问题**

https://stackoverflow.com/questions/1063438/sqlite3-and-multiple-processes
