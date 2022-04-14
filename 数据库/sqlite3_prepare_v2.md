sqlite3_exec 实际上是将编译，执行进行了封装，与之等价的一组函数是 sqlite3_prepare_v2(), sqlite3_step() 和 sqlite3_finalize()。

sqlite3_prepare_v2() 编译 SQL 语句生成 VDBE 执行码，sqlite3_step() 执行，sqlite3_finalize() 关闭语句句柄，释放资源。两种方式，都可以通过调用 sqlite3_changes(pdb)，得到语句影响的
行数。

## 两种方式比较

- sqlite3_exec 方式接口使用很简单，实现同样的功能，比 sqlite3_perpare_v2 接口代码量少。
- sqlite3_prepare 方式更高效，因为只需要编译一次，就可以重复执行N次。
- sqlite3_prepare 方式支持参数化 SQL。

鉴于两种方式的差异，对于简单的 PRAGMA 设置语句(PRAGMA cache_size=2000)，事务设置语句(BEGIN TRANSACTION,COMMIT,ROLLBACK)使用 sqlite3_exec 方式，更简单；而对于批量的更新、查询语句，
则使用 sqlite3_prepare 方式，更高效。

注意，sqlite3_prepare_v2()、sqlite3_step()、sqlite3_finalize() 三个函数必须是顺序执行的。不能没有 sqlite3_finalize() 就又执行 sqlite3_prepare_v2()。比如，数据库打开后，按照 
sqlite3_prepare_v2()、
sqlite3_step()、sqlite3_prepare_v2()、sqlite3_step()……如此循环，最后才有一个 sqlite3_finalize()。程序执行后不提示错误，但是查看 table 的 row 可以知道未能插入数据。解决办法就是
sqlite3_step 后再加一个 sqlite3_finalize。

## 参考

- https://www.cnblogs.com/cchust/p/5121559.html
