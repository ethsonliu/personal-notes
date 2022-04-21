https://www.cplusplus.com/reference/cassert/assert/

需要注意的点：

- 断言失败就会调用 `abort` 函数，并且打印错误输出
- 禁止 assert 断言，把 `#define NDEBUG` 写在 `#include <assert.h>` 的前面
- assert 就是为了方便调试判断，别无其它含义，比如网上说的什么 assert 等价于 I make sure
