```c++
// 用法一
std::unique_lock<std::mutex> lock(mtx);
cv.wait(mtx, [this]
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

