转自：<http://kaiyuan.me/2018/05/08/sfinae/>

最近在看 Facebook 开源的 C++ 组件库 [folly](https://github.com/facebook/folly) 的源码，不仅收获了很多 Modern C++ 的编程技巧(e.g. type_traits，meta-programming)，也了解了很多高效数据结构的设计思想(e.g. FBString，Atomic Data Structures)。不得不说，folly 与 Leveldb 一样具有很高的源码质量，尽管采用了较多的高级特性及平台相关特性，但是阅读起来并不会因代码风格而吃力。今天这篇文章，我们来介绍一下在 folly 中被大量使用的现代 C++ 模板特性 -- ***Substitution failure is not an error (SFINAE)***。

在开始今天的主题之前，先思考一下如下两个通常可能会遇到的问题：

1. 如何在编译期间判断某个类型 T 是否是 class 类型或者判断某个 class T 是否有指定的成员变量或成员函数？
2. 如何在函数编译期间根据特定的条件来选择启用或禁用特定的重载？

<!--more-->

## 什么是 SFINAE

"Substitution failure is not an error" 直译过来就是『替换失败不是错误』。 [cppreference](http://zh.cppreference.com/w/cpp/language/sfinae) 给出了该原则的精确定义：

> 在函数模板的重载决议中：为模板形参替换推导类型失败时，从*重载集*抛弃特化，而非导致编译错误。

这个定义看起来有些抽象，理解它的关键在于弄清楚**替换**，**失败**和**错误**这三个关键字的含义。**替换**指的是函数模板形参被替换为实参的整个过程，可以发生于函数模板的模板形参，函数的参数类型以及返回类型。我们通过如下例子来具体说明替换发生的场景：

```cpp
class Bar {
    typedef int type;
};

// example 1
template <typename T, typename = typename T::type*>
int foo1(int x) {
    return x;
}

foo1<Bar>(12);  // 模板形参替换 

// example 2
template <int M, int N> 
void foo2(char(*)[M <= N] = 0) {  
}

foo2<10, 12>();  // 函数参数类型替换

// example 3
template <typename T>
typename std::enable_if<std::is_integral<typename T::type>::value>::type 
    foo3() {
    typename T::type x = 1;
    return x;
}

std::cout << foo3<Bar>();  // 函数返回值类型替换
```

例 1 展示了对于函数模板形参的替换。`foo1` 函数的第二个模板参数为匿名形参，且通过域解析运算符解析T 类型中的 `type` 类型，因此调用 `foo1<Bar>()` 可以将 `T` 替换成为 `Bar` 从而实例化成为 `foo<Bar, Bar::type*>()`。

例 2 展示对于函数参数的替换。这里可能需要解释一下 `char(*)[N]` 语法，其表示一个指向 N 个单位的一维数组的指针。如果 `M <= N` 满足，则 `char(*)[1]` 是一个合法的指向长度为 1 的数组指针；而如果 `M > N`，则得到 `char(*)[0]` 在 C++ 中不合法。因此本例中 `foo2<10, 12>()` 可以顺利编译通过。

例 3 展示了对于函数返回值的替换。这里使用了 C++11 type_traits 语法 `enable_if` 和 `is_integral`。由于 `Bar::type` 为 `int` 类型，因此 `is_integral<Bar::type>` 将返回 `true`，即 `foo3<Bar>` 可以成功实例化。

这三个例子粗略展示了函数模板『替换』发生的场景，更加严谨具体的替换规则请参考 [cppreference](http://zh.cppreference.com/w/cpp/language/sfinae)。有了这三个替换成功的案例参考，『失败』就很好理解了。非正式的说，当传递给函数模板的模板实参不能完成相应的实参推导，那么替换就会失败。例如我们企图调用：

```cpp
foo2<12, 10>();
```

`foo2<M, N>` 将会被实例化为 `foo2(char(*)[0])`，由于 0 长度数组指针不合法，编译器会立刻向我们报告错误：

```
error: no matching function for call to 'foo2'
candidate template ignored: substitution failure [with M = 12, N = 10]
```

既然替换失败后编译器会报告错误信息，为什么我们还要来讨论『替换失败不是错误』原则呢？事实上，上面这三个例子非常极端，每个模板函数都仅仅具有一种函数原型，即对应函数的重载集只有一个元素。一旦不能匹配到该函数模板，编译器没有其他选择，就只能报告错误(error)。如果我们为上例中的 `foo2` 函数增加一个如下的重载，并再次调用 `foo2<12, 10>()`：

```cpp
template <int M, int N> 
void foo2(char(*)[M > N] = 0) {  
}

foo2<12, 10>(); // 实例化
```

执行编译之后，你会发现编译器智能地把 `foo2<12, 10>` 匹配成为 `foo2(char(*)[1])` 而不是 `foo2(char(*)[0])`，从而避免报告错误。整个替换的大致如下：编译器首先尝试(按照词序)去匹配函数模板，由于 `M 和 N` 不满足 `M <= N` 的检查导致本次替换失败；此时编译器并不会报告错误，而是去尝试匹配重载集里的另一个原型，匹配成功后编译通过。

到此，SFINAE 的含义应该已经比较清楚：编译器在将函数模板形参替换成实参的过程中，如果针对某个函数模板产生了不合法的代码，其不会直接抛出错误信息，而是继续尝试去匹配其他可能的重载函数。


## SFINAE 有什么用

现在我们回过头去看一下本文开头所提出的两个问题。

第一个问题：*如何在编译期间判断某个类型 T 是否是 class 或者判断某个类是否有指定的成员变量或成员函数*？这个问题可以说是 SFINAE 最典型的应用。套用 SFINAE 原则，我们需要两个返回不同值的模板函数，当传入 Class 作为模板实参时匹配到其中一个函数模板，而传入其他实参时匹配到另一个。这样我们就可以根据返回值将 class 与其他类型区分开。

这个问题看起来是不是非常简单？其实不然。由于我们要在**编译期间**而不是运行期间根据返回值来区分传给函数模板的实参，所以我们还需要一些其他的手段，比如 `sizeof` 操作符。`sizeof` 操作符可以在编译期间返回指定的类型占用的空间大小，利用这一点，我们可以轻松的写出如下代码：

```cpp
template<typename T>
class IsClass {
  private:
    typedef char One;
    typedef struct { char a[2]; } Two;
    template<typename C> static One test(int C::*);
    template<typename C> static Two test(...);
  public:
    static bool value = sizeof(IsClass<T>::test<T>(NULL)) == 1;
};

bool is_class = IsClass<Test>::value;
```

这段代码虽然不难理解，但仍有两个细节需要说明：

1. 注意 ellipse 符号 `...` 操作符在 C++ 中的应用。有经验的编程者都知道，在 C 语言标准中，`...` 是不能作为唯一的参数放在函数参数列表中的，也就是说形如 `void test(...)` 的函数在 C 语言中是无法通过编译的（思考一下为什么？）。然而在 C++ 语言中，此类型的用法符合语言标准，并且仅仅用在 SFINAE 场景下来匹配任意参数。
2. `int C::*` 这个语法看起来似乎非常奇怪，它代表了指向 class C 的成员的指针。具体这种指针的用处，可以参考 [Pointer to a member in C++](https://stackoverflow.com/questions/670734/c-pointer-to-class-data-member)。这里我们使用这种语法来触发域解析符的使用，从而确定某个类型 `T` 能否可以使用 `int T::*` 这种形式。

上述的代码中，如果得到 `is_class` 的值为 `true`，那么表明类型 `Test` 为一个 class 类型。我们也可以使用 C++11 的 `constexpr` 关键字来简化这段代码的逻辑：

```cpp
template<typename T>
class IsClass {
  private:
    template<typename C> static constexpr bool test(int C::*) {
        return true;
    }
    template<typename C> static constexpr bool test(...) {
        return false;
    }
  public:
    static constexpr bool value = IsClass<T>::test<T>(nullptr);
};
```

接下来看第二个问题：*如何在函数编译期间根据特定的条件来选择启用或禁用特定的重载*？相比于上个问题，这个问题的要求更加灵活：在编译期间根据条件来完成相应的模板匹配。在 C++11 之前，我们可能需要大费周章地来达到这个目的，所幸 C++11 提供的 type_traits (e.g. enable_if, conditional) 已经可以帮助我们大大减少工作量和维护成本。关于 type_traits 的语义，这里不多赘述，可以自行查阅资料。

举个简单的例子，假如我们想通过一个函数 `ToString` 来把 `int`, `double`, `string` 类型转换成为一个 `string` 类型，通常我们需要这么来处理：

```cpp
template <typename T>
string ToString(T& t) {
    if(typeid(T) == typeid(int) || typeid(T) == typeid(float) ||
        typeid(T) == typeid(double)) {
        return std::to_string(t);
    }
    else if(typeid(T) == typeid(string) {
        return t;
    }
}
```

这样写是不是非常麻烦？假如以后还要添加 `long`，`float` 类型，我们是否需要去 `ToString` 函数内部来增加新 `typeid` 的判断？再假如后面还要添加数组类型我们又该怎么办？利用 C++11 的 `enable_if` 以及 SFIANE 的原则，优雅的实现方式如下：

```cpp
template <typename T>
typename std::enable_if<std::is_arithmetic<T>::value, string>::type
    ToString(T& t) {
    return std::to_string(t);
}

template <typename T>
typename std::enable_if<std::is_array<T>::value, string>::type
    ToString(T& t) {
    return ArrayToString(T);  // ArrayToString 需要自行实现
}

template <typename T>
typename std::enable_if<std::is_same<T, string>::value, string>::type
    ToString(T& t) {
    return t;
}
```

令人惊讶的是，这几个函数竟然仅仅利用返回值不同而实现了函数重载，这与我们最初学习的 C++ 重载规则产生了矛盾。这正是 `std:enable_if` 的厉害之处，其可以帮助我们实现更加强大的重载机制，包括仅仅通过返回值来实现重载。当然，我们也可以通过模板参数或者函数形参来达到同样的目的：

```c++
// 通过模板参数来实现 SFINAE
template <typename T,
    typename = typename std::enable_if<std::is_arithmetic<T>::value>::type>
string ToString(T& t) {
    return std::to_string(t);
}

// 通过函数的参数来实现 SFINAE
template <typename T>
string ToString(T& t, 
    typename std::enable_if<std::is_arithmetic<T>::value>::type* = 0) {
    return std::to_string(t);
}
```

尽管这三种实现方式都可以达到目的，我们还是推荐使用重载函数模板参数的方式。具体的原因可以参考 [enable_if in function signatures](https://stackoverflow.com/questions/14600201/why-should-i-avoid-stdenable-if-in-function-signatures)。而 Scott Meyers 在他关于 EC++11 的 [post](http://scottmeyers.blogspot.de/2013/01/effective-c11-content-and-status.html) 中也支持了这种说法。

## 总结

这篇文章我们通过两个小例子来介绍了 C++ 函数模板匹配过程中的 SFINAE 原则及其作用。写作的过程中我越来越感觉到，很多时候 C++ 底层机制默默地帮助我们做了很多事情，而我们却浑然不知，这不应该是一个 C++ programmer 应有的状态。下一次有时间，我们再来详细学习一下本文中未提到的 C++ 模板函数的重载机制。


 















