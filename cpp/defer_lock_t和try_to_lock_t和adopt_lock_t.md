- defer_lock_t	do not acquire ownership of the mutex
- try_to_lock_t	try to acquire ownership of the mutex without blocking
- adopt_lock_t	assume the calling thread already has ownership of the mutex

https://en.cppreference.com/w/cpp/thread/lock_tag_t

acquire ownership 的意思可以理解成 lock()
