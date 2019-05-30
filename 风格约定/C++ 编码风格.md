## 目录

- [头文件](#头文件)
  - [define 保护](#define-保护)
  - [前置声明](#前置声明)
  - [include 的路径及顺序](#include-的路径及顺序)
- [作用域](#作用域)
- [参考](#参考)


## 头文件

### define 保护

所有头文件都应该使用`#define`来防止被多次包含，命名格式是：`<PROJECT>_<PATH>_<FILE>_H`。例如`MYPROJECT_SRC_WIDGET_SIMPLE_WIDGET_H`。

工程名如果过长，可以适当简写。路径需要详细，不要遗漏。

### 前置声明

尽可能地避免使用前置声明。使用`#include`包含需要的头文件即可。

前置声明的优缺点如下，

- 优点：

  前置声明能够节省编译时间，多余的`#include`会迫使编译器展开更多的文件，处理更多的输入。

  前置声明能够节省不必要的重新编译的时间。 `#include`使代码因为头文件中无关的改动而被重新编译多次。

- 缺点：

  前置声明隐藏了依赖关系，头文件改动时，用户的代码会跳过必要的重新编译过程。（注：不懂这是什么意思？）

  前置声明可能会被库的后续更改所破坏。前置声明函数或模板有时会妨碍头文件开发者变动其`API`。例如扩大形参类型，加个自带默认参数的模板形参等等。

  前置声明来自命名空间`std::`的`symbol`时，其行为未定义。

极端情况下，用前置声明代替`includes`甚至都会暗暗地改变代码的含义：

```c++
// b.h:
struct B {};
struct D : B {};

// good_user.cc:
#include "b.h"
void f(B*);
void f(void*);
void test(D* x) { f(x); }  // calls f(B*)
```

如果`#include`被 B 和 D 的前置声明替代，`test()`就会调用`f(void*)` 。

如果仅仅为了能前置声明而重构代码（比如用指针成员代替对象成员），会使代码变得更慢更复杂，得不偿失。

### include 的路径及顺序

1. The prototype/interface header for this implementation (ie, the .h/.hh file that corresponds to this .cpp/.cc file).
2. Other headers from the same project, as needed.
3. Headers from other non-standard, non-system libraries (for example, Qt, Eigen, etc).
4. Headers from other "almost-standard" libraries (for example, Boost)
5. Standard C++ headers (for example, iostream, functional, etc.)
6. Standard C headers (for example, cstdint, dirent.h, etc.)

为什么不采用 Google 推荐的呢？第一，我没看懂它那样的用意；第二，上面这六条大致是符合我自己的思路的。

## 作用域



## 参考

- [Google 开源项目风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/)
- [Stackoverflow. C/C++ include header file order](https://stackoverflow.com/questions/2762568/c-c-include-header-file-order)
