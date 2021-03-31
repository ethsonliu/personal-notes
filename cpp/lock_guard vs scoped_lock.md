lock_guard 是 C++11 出的，scoped_lock 是 C++17 出的。

scoped_lock 其实是 lock_guard 的一个更佳版本。

So my advice is to use the simplest tool for the job:

- lock_guard if you need to lock exactly 1 mutex for an entire scope.
- scoped_lock if you need to lock a number of mutexes that is not exactly 1.
- unique_lock if you need to unlock within the scope of the block (which includes use with a condition_variable).

参考：

- <https://stackoverflow.com/questions/43019598/stdlock-guard-or-stdscoped-lock>
