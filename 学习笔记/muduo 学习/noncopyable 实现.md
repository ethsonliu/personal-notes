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

为什么上面开头的代码要用私有继承呢？这里有相关的讨论，见 <https://stackoverflow.com/questions/5654330/privately-or-publicly-inherit-from-boostnon-copyable>，因为 `boost::noncopyable` 没有 virtual 析构函数。

其中 protected 构造函数和析构函数是为了强调这只能用作基类，详见 <https://www.boost.org/doc/libs/1_63_0/libs/core/doc/html/core/noncopyable.html>。

那么 `boost::noncopyable` 的优点就是这个么？或者说有缺点么？这个是 Stack Overflow 上对此的讨论，见 <https://stackoverflow.com/questions/7823990/what-are-the-advantages-of-boostnoncopyable>。



参考：

- <https://www.boost.org/doc/libs/1_63_0/libs/core/doc/html/core/noncopyable.html>
- <http://bajamircea.github.io/coding/cpp/2017/02/22/noncopyable-adl.html>
- <https://stackoverflow.com/questions/7823990/what-are-the-advantages-of-boostnoncopyable>
- <https://stackoverflow.com/questions/5654330/privately-or-publicly-inherit-from-boostnon-copyable>
