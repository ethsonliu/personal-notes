enable_shared_from_this 是一个模板类，定义于头文件 <memory>，其原型为：

```c++
template <class T>
class enable_shared_from_this;
```

std::enable_shared_from_this 能让一个对象（假设其名为 t ，且已被一个 std::shared_ptr 对象 pt 管理）安全地生成其他额外的 std::shared_ptr 实例（假设名为 pt1, pt2, ... ） ，它们与 pt 共享对象
t 的所有权。

若一个类 T 继承 std::enable_shared_from_this<T> ，则会为该类 T 提供成员函数：shared_from_this 。 当 T 类型对象 t 被一个为名为 pt 的 std::shared_ptr<T> 类对象管理时，调用 
T::shared_from_this 成员函数，将会返回一个新的 std::shared_ptr<T> 对象，它与 pt 共享 t 的所有权。

为何会出现这种使用场合呢？

因为在异步调用中，存在一个保活机制，异步函数执行的时间点我们是无法确定的，然而异步函数可能会使用到异步调用之前就存在的变量。为了保证该变量在异步函数执期间一直有效，我们可以传递一个指向自身的 
share_ptr 给异步函数，这样在异步函数执行期间 share_ptr 所管理的对象就不会析构，所使用的变量也会一直有效了（保活）。

使用范例，

```c++
#include <memory>
#include <iostream>
 
struct Good: std::enable_shared_from_this<Good> // note: public inheritance
{
    std::shared_ptr<Good> getptr() {
        return shared_from_this();
    }
};
 
struct Bad
{
    std::shared_ptr<Bad> getptr() {
        return std::shared_ptr<Bad>(this);
    }
    ~Bad() { std::cout << "Bad::~Bad() called\n"; }
};
 
int main()
{
    // Good: the two shared_ptr's share the same object
    std::shared_ptr<Good> gp1 = std::make_shared<Good>();
    std::shared_ptr<Good> gp2 = gp1->getptr();
    std::cout << "gp2.use_count() = " << gp2.use_count() << '\n';
 
    // Bad: shared_from_this is called without having std::shared_ptr owning the caller 
    try {
        Good not_so_good;
        std::shared_ptr<Good> gp1 = not_so_good.getptr();
    } catch(std::bad_weak_ptr& e) {
        // undefined behavior (until C++17) and std::bad_weak_ptr thrown (since C++17)
        std::cout << e.what() << '\n';    
    }
 
    // Bad, each shared_ptr thinks it's the only owner of the object
    std::shared_ptr<Bad> bp1 = std::make_shared<Bad>();
    std::shared_ptr<Bad> bp2 = bp1->getptr();
    std::cout << "bp2.use_count() = " << bp2.use_count() << '\n';
} // UB: double-delete of Bad
```

（可能的）输出如下，

```plaintext
gp2.use_count() = 2
bad_weak_ptr
bp2.use_count() = 1
Bad::~Bad() called
Bad::~Bad() called
*** glibc detected *** ./test: double free or corruption
```

为什么中间那个会抛异常呢？

这要从 enable_shared_from_this 这个类说起，这个类的核心在于其内部维护了一个 weak_ptr，通过 weak_ptr 可以把没有关系的 shared_ptr 联系起来，增加引用计数。但是，weak_ptr 的初始化操作在 
std::make_shared 函数里面完成，并不是在 enable_shared_from_this 类的构造中完成，所以凡是没调用 std::make_shared 的对象，均会因为 weak_ptr 没初始化导致异常。

源码参见，

1. [boost/boost/smart_ptr/enable_shared_from_this.hpp](https://code.woboq.org/boost/boost/boost/smart_ptr/enable_shared_from_this.hpp.html)
2. [boost/boost/smart_ptr/shared_ptr.hpp](https://code.woboq.org/boost/boost/boost/smart_ptr/shared_ptr.hpp.html)
3. [boost/boost/smart_ptr/weak_ptr.hpp](https://code.woboq.org/boost/boost/boost/smart_ptr/weak_ptr.hpp.html)

## 参考：

1. [C++ 11 新特性之十：enable_shared_from_this](https://blog.csdn.net/caoshangpa/article/details/79392878)
2. <https://en.cppreference.com/w/cpp/memory/enable_shared_from_this>
