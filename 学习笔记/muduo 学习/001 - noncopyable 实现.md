**文件：noncopyable.h**

`boost::noncopyable` 允许程序轻松地实现一个禁止拷贝的类，使用时只需包含头文件 `<boost/noncopyable.hpp>` 即可。

```c++
class A : boost::noncopyable // 私有继承
{
};

int main()
{
    A a1;
    A a2 = a1;

    return 0;
}
```

运行报错，

```plaintext
error: C2280: “A::A(const A &)”: 尝试引用已删除的函数
```

`boost::noncopyable` 的实现原理很简单，下面是 boost 实现源码，地址在 [boost/boost/core/noncopyable.hpp](https://code.woboq.org/boost/boost/boost/core/noncopyable.hpp.html)，

```c++
namespace boost {
//  Private copy constructor and copy assignment ensure classes derived from
//  class noncopyable cannot be copied.
//  Contributed by Dave Abrahams
namespace noncopyable_  // protection from unintended ADL
{
  class noncopyable
  {
  protected:
#if !defined(BOOST_NO_CXX11_DEFAULTED_FUNCTIONS) && !defined(BOOST_NO_CXX11_NON_PUBLIC_DEFAULTED_FUNCTIONS)
      BOOST_CONSTEXPR noncopyable() = default;
      ~noncopyable() = default;
#else
      noncopyable() {}
      ~noncopyable() {}
#endif
#if !defined(BOOST_NO_CXX11_DELETED_FUNCTIONS)
      noncopyable( const noncopyable& ) = delete;
      noncopyable& operator=( const noncopyable& ) = delete;
#else
  private:  // emphasize the following members are private
      noncopyable( const noncopyable& );
      noncopyable& operator=( const noncopyable& );
#endif
  };
}
typedef noncopyable_::noncopyable noncopyable;
} // namespace boost
```

1. 为什么上面开头的代码要用私有继承呢？
    
    boost::noncopyable does not declare a virtual destructor, i.e. is not designed to be the base of public inheritance chain. Always inherit from it privately.

2. 构造函数和析构函数为什么用 protected 修饰？

    是为了强调这只能用作基类，你不可以用它来定义对象。

3. 那么 `boost::noncopyable` 的优点就是这个么？或者说有缺点么？为什么不可以直接用 delete 来修饰呢？

    `boost::noncopyable` 的好处就是可以很明显地提醒别人这是不可复制的，但在多继承下，会占用多余的空间，就个人而言，直接使用 delete 比较好，当然可以退一步，定义一个宏，比如 NONCOPYABLE 用来提醒别人也未尝不可。

参考：

- <https://www.boost.org/doc/libs/1_63_0/libs/core/doc/html/core/noncopyable.html>
- <http://bajamircea.github.io/coding/cpp/2017/02/22/noncopyable-adl.html>
- <https://stackoverflow.com/questions/7823990/what-are-the-advantages-of-boostnoncopyable>
- <https://stackoverflow.com/questions/5654330/privately-or-publicly-inherit-from-boostnon-copyable>
