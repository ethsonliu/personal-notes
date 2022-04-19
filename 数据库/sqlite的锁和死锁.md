
## 锁的介绍







## 预防死锁

### 事务类型

```
BEGIN [ DEFERRED | IMMEDIATE | EXCLUSIVE ] TRANSACTION
```

默认是 DEFERRED。

- DEFERRED，事务开始时并不获取任何锁，直到它需要锁的时候才加锁，而且 BEGIN 语句本身也不会做什么事情，它开始处于 UNLOCK 状态。
- IMMEDIATE，事务开始时会试着获取 RESERVED LOCK，如果成功，则事务处于 RESERVED 锁状态，以保证后续没有别的连接可以写数据库，但是，别的连接仍可以对数据库进行读操作； RESERVED LOCK 会阻止其它的连接以 BEGIN IMMEDIATE 或者 BEGIN EXCLUSIVE 命令开始事务，SQLite 会返回 SQLITE_BUSY 错误；但可以以 BEGIN DEFERRED 命令开始事务。事务开始成功后，就可以对数据库进行修改操作；当 COMMIT 时，如果返回 SQLITE_BUSY 错误，这意味着还有其它的读事务没有完成，得等它们执行完后才能提交事务。
- EXCLUSIVE，事务开始时会试着获取 EXCLUSIVE LOCK。这与 IMMEDIATE 类似，但是一旦成功，EXCLUSIVE 事务保证没有其它的连接（其它事务读库都不行，因为获取不到 SHARED LOCK），所以此事务就可对数据库进行读写操作了。

使用指导，

- Use just BEGIN TRANSACTION (or explicitly BEGIN DEFERRED TRANSACTION) when you only expect to read. Writes could possibly fail, forcing you to rollback and retry the entire transaction again.
- Use BEGIN IMMEDIATE TRANSACTION when you expect to maybe write at some point. This will block all other writers and all other immediate maybe-writers.
- BEGIN EXCLUSIVE TRANSACTION will immediately block until all other locks are released. I have no idea why anyone would want this. Possibly to prepare for some data which needs to be written to disk as quickly as possible once it arrives? EDIT: It seems to be the only way to prevent timeouts at arbitrary points after beginning a transaction.

###  程序控制

通过保证线程安全来防止死锁。

- 单线程模式，使用一个专门线程访问数据库
- 单线程模式，使用一个线程队列访问数据库（队列里线程共用一个数据库连接）

## 参考

- https://my.oschina.net/u/587236/blog/129022
