## 目录

- [头文件](#头文件)
  - [define 保护](#define-保护)
  - [前置声明](#前置声明)
  - [include 的路径及顺序](#include-的路径及顺序)
- [作用域](#作用域)
  - [命名空间](#命名空间)
  - [匿名命名空间和静态变量](#匿名命名空间和静态变量)
  - [非成员函数-静态成员函数-全局函数](#非成员函数-静态成员函数-全局函数)
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

### 命名空间

- 在编写自己的库的时候，需要加上命名空间，命名格式参考下面的【命名约定】，防止开发者使用过程中造成命名冲突。
- 不应该使用 using 指示 引入整个命名空间的标识符号。
- 内联命名空间很容易令人迷惑，毕竟其内部的成员不再受其声明所在命名空间的限制。内联命名空间只在大型版本控制里有用，主要用来保持跨版本的 ABI 兼容性。

### 匿名命名空间和静态变量

在`.cpp`文件中定义一个不需要被外部引用的变量时，可以将它们放在匿名命名空间或声明为`static`。但是不要在`.h`文件中这么做。

所有置于匿名命名空间的声明都具有内部链接性，函数和变量可以经由声明为`static`拥有内部链接性，这意味着你在这个文件中声明的这些标识符都不能在另一个文件中被访问。即使两个文件声明了完全一样名字的标识符，它们所指向的实体实际上是完全不同的。

所以推荐和鼓励在`.cpp`中对于不需要在其他地方引用的标识符使用内部链接性声明，但是不要在`.h`中使用。

### 非成员函数-静态成员函数-全局函数



## 参考

- [Google 开源项目风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/)
- [Stackoverflow. C/C++ include header file order](https://stackoverflow.com/questions/2762568/c-c-include-header-file-order)
