- lock_guard，c++11，提供的接口很简单，只有构造函数和析构函数，且只能对一个 mutex 操作，不可复制，不可移动。
- unique_lock，c++11，除了 lock_guard 提供的功能外，还可以再对 mutex 进行 lock 和 unlock，参照例子 https://en.cppreference.com/w/cpp/thread/unique_lock/lock ，这也就意味着 unique_lock 内部需要保存 mutex 的状态，还需要在析构的时候判断是否需要 unlock，总体代价比 lock_guard 大。只能对一个 mutex 操作，不可复制，可移动（所以可以作为函数的返回值，也可以放到STL的容器中，但前提是你可能得显示调用 std::move）。
- shared_lock，c++14，搭配 shared_mutex，也就是读写锁使用。
- scoped_lock，c++17，提供的接口和lock_guard一样很简单，和lock_guard的区别就是它可以对多个mutex操作。

So my advice is to use the simplest tool for the job:

- lock_guard if you need to lock exactly 1 mutex for an entire scope.
- scoped_lock if you need to lock a number of mutexes that is not exactly 1.
- unique_lock if you need to unlock within the scope of the block (which includes use with a condition_variable).

参考：

- <https://stackoverflow.com/questions/43019598/stdlock-guard-or-stdscoped-lock>
