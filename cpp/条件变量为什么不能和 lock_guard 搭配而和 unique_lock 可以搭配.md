```c++
// 用法一
std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock, [this]
{
    return !(!this->m_stop && this->m_tasksQue.empty());
});

// 用法二
std::lock_guard<std::mutex> lock(mtx);
cv.wait(mtx, [this]
{
    return !(!this->m_stop && this->m_tasksQue.empty());
});
```
condition_variable::wait 内部会检查 mtx 的所有权，而 lock_guard 会把 mtx 的所有权拿走导致无法在 wait 内部进行 unlock 等操作。

参考：

- <https://stackoverflow.com/questions/13099660/c11-why-does-stdcondition-variable-use-stdunique-lock>
