## 目录

- [头文件](#头文件)
  - [define 保护](#define-保护)
  - [前置声明](#前置声明)
  - [include 头文件的顺序](#include-头文件的顺序)
- [作用域](#作用域)
  - [命名空间](#命名空间)
  - [匿名命名空间和静态变量](#匿名命名空间和静态变量)
  - [全局函数](#全局函数)
  - [局部变量](#局部变量)
  - [静态和全局变量](#静态和全局变量)
- [类](#类)
  - [构造函数](#构造函数)
  - [隐式类型转换](#隐式类型转换)
  - [结构体和类](#结构体和类)
  - [声明顺序](#声明顺序)
- [函数](#函数)
  - [参数顺序](#参数顺序)
  - [编写简短函数](#编写简短函数)
  - [缺省参数](#缺省参数)
- [命名约定](#命名约定)
  - [文件命名](#文件命名)
  - [类型命名](#类型命名)
  - [变量命名](#变量命名)
  - [函数命名](#函数命名)
  - [命名空间命名](#命名空间命名)
  - [枚举命名](#枚举命名)
  - [宏命名](#宏命名)
- [注释](#注释)
  - [注释风格](#注释风格)
  - [文件注释](#文件注释)
  - [类注释](#类注释)
  - [函数注释](#函数注释)
  - [块内注释](#块内注释)
  - [TODO 注释](#TODO-注释)
- [其它编程建议](#其它编程建议)
  - [include 头文件的路径](#include-头文件的路径)
  - [避免隐式转换和比较](#避免隐式转换和比较)
  - [智能指针代替原生指针](#智能指针代替原生指针)
  - [尽可能地避免产生内存碎片](#尽可能地避免产生内存碎片)
  - [尽量使用前置自增语句](#尽量使用前置自增语句)
  - [尽可能地使用 assert 语句](#尽可能地使用-assert-语句)
  - [避免操作符的短路问题](#避免操作符的短路问题)
  - [尽可能使用无异常的函数](#尽可能使用无异常的函数)
  - [不要用尤达条件式](#不要用尤达条件式)
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

### include 头文件的顺序

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
- 如果是在开发一个应用的话，就是只是调库，而不是造轮子，没必要加命名空间，因为你在做的已经是最上层的应用了。
- 不应该使用 using 指示 引入整个命名空间的标识符号。
- 内联命名空间很容易令人迷惑，毕竟其内部的成员不再受其声明所在命名空间的限制。内联命名空间只在大型版本控制里有用，主要用来保持跨版本的 ABI 兼容性。

### 匿名命名空间和静态变量

在`.cpp`文件中定义一个不需要被外部引用的变量时，可以将它们放在匿名命名空间或声明为`static`。但是不要在`.h`文件中这么做。

所有置于匿名命名空间的声明都具有内部链接性，函数和变量可以经由声明为`static`拥有内部链接性，这意味着你在这个文件中声明的这些标识符都不能在另一个文件中被访问。即使两个文件声明了完全一样名字的标识符，它们所指向的实体实际上是完全不同的。

所以推荐和鼓励在`.cpp`中对于不需要在其他地方引用的标识符使用内部链接性声明，但是不要在`.h`中使用。

### 全局函数

尽量别使用全局函数，能以`static`内部链接的就必须内部链接。

### 局部变量

将函数变量尽可能置于最小作用域内，并在变量声明时进行初始化。

我们提倡在尽可能小的作用域中声明变量，离第一次使用越近越好，这使得代码浏览者更容易定位变量声明的位置。

有一个例外，如果变量是一个对象，每次进入作用域都要调用其构造函数，每次退出作用域都要调用其析构函数，这会导致效率降低。例如下面的代码：

```c++
// 低效的实现
for (int i = 0; i < 1000000; ++i)
{
    Foo f; // 构造函数和析构函数分别调用 1000000 次!
    f.DoSomething(i);
}

// 高效的实现
Foo f;  // 构造函数和析构函数只调用 1 次
for (int i = 0; i < 1000000; ++i)
{
    f.DoSomething(i);
}
```

### 静态和全局变量

禁止定义静态储存周期非 POD 变量。

静态生存周期的对象，即包括了全局变量，静态变量，静态类成员变量和函数静态变量。

原生数据类型 (POD : Plain Old Data): 即 int, char 和 float, 以及 POD 类型的指针、数组和结构体。

按我的理解，面向对象编程中，静态和全局变量都是可以放在类内的，杜绝一切位于全局的非 POD 变量就对了，少费点脑子。

## 类

### 构造函数

不要在构造函数中调用虚函数, 也不要在无法报出错误时进行可能失败的初始化。

### 隐式类型转换

不要定义隐式类型转换，对于转换运算符和单参数构造函数，请使用`explicit`关键字。

### 结构体和类

仅当只有 POD 数据成员时使用`struct`，其它一概使用`class`。

在 C++ 中`struct`和`class`关键字几乎含义一样，我们为这两个关键字添加我们自己的语义理解，以便为定义的数据类型选择合适的关键字。

为了和 STL 保持一致, 对于仿函数等特性可以不用`class`而是使用`struct`。

### 声明顺序

类定义一般应以`public:`开始，后跟`protected:`，最后是`private:`。

## 函数

### 参数顺序

函数的参数顺序为: 输入参数在先，后跟输出参数。

### 编写简短函数

长函数有时很方便，但一旦重构起来，会很烦，头找不到尾，尾找不到头，变量满天飞的感觉。

如果可以，分割它们。

### 缺省参数

只允许在非虚函数中使用缺省参数。

## 命名约定

### 文件命名

文件名要全部小写，单词之间以`_`连接。

文件名对该文件的描述要清晰，不要用简写。

例如`my_useful_class.cpp`。

### 类型命名

类型名称的每个单词首字母均大写，不包含下划线: `MyExcitingClass`，`MyExcitingEnum`。

所有类型命名，例如，类，结构体，类型定义 (typedef)，枚举，类型模板参数，均使用相同约定。

```c++
// 类和结构体
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// 类型定义
typedef hash_map<UrlTableProperties *, string> PropertiesMap;

// using 别名
using PropertiesMap = hash_map<UrlTableProperties *, string>;

// 枚举
enum UrlTableErrors { ...
```

### 变量命名

普通变量命名，首字母小写，后面的单词首字母大写，例如：

```c++
int tablesNumber;
string tableName;
```

类数据成员，前缀加`m_`，剩下的和普通变量命名一致。例如：

```c++
class TableInfo
{
  ...
 private:
  string m_tableName;
  Pool<TableInfo>* m_pool;
};
```

结构体变量，遵循普通变量命名，例如：

```c++
struct UrlTableProperties
{
  string name;
  int numEntries;
};
```

静态变量，不管位于哪个作用域，即类（或者结构体）的静态变量，函数内静态变量等等，命名规则一律和普通变量命名一致。

常量，一律大写，例如：

```c++
const int DAYS_IN_A_WEEK = 7;
```

### 函数命名

常规函数，单词首字母都大写，例如：

```c++
void MyExcitingFunction()
{
}
```

类内的函数，不管是普通的类内函数，还是类内静态函数，除第一个单词的首字母小写外，后面的单词首字母都大写。

```c++
class MyClass
{
  public:
    void getName() const;
};
```

### 命名空间命名

命名空间每个单词首字母都大写，例如，

```c++
namespace WebSearch
{
}
```

### 枚举命名

和普通变量命名一致，全部以`enum class`代替`enum`，不准用`enum`定义枚举。例如：

```c++
enum class EnumType
{
  appleType,
  bananaType,
  watermelonType
};
```

### 宏命名

全部大写，单词间以下划线连接，例如：

```c++
#define ROUND(x) ...
#define PI_ROUNDED 3.0
```

## 注释

### 注释风格

统一以 Javadoc `/**  */`风格，具体参考 jdk，这里只做简单举例和描述。以下的例子都是基于上层开发，注释只是为了方便阅读代码，而不是生成文档。

### 文件注释

注释的作用是版权说明、描述所执行的功能、作者等等。但描述所执行的功能这个，一般都做在其它地方了，这里不需要写。版权说明和作者这些，现在暂无这方面的考虑，此处以后再进行更新。

### 类注释

```c++
/**
 * Utility methods for packing/unpacking primitive values in/out of byte arrays
 * using big-endian byte ordering.
 */
class Bits
{

}
```

### 函数注释

如果有的参数和返回值易于分辨，就不必用`@param`和`@return`加以说明。

```c++
/**
 * Retrieves the contents of the SQL ARRAY value designated
 * by this.
 *
 * @param     checkKind
 *            the kind of bounds check, whose name may correspond
 *            to the name of one of the range check methods, checkIndex.
 *
 * @param     args
 *            the out-of-bounds arguments that failed the range check.
 *            If the checkKind corresponds a the name of a range check method.
 *
 * @return    an array in the Java programming language that contains
 *            the ordered elements of the SQL ARRAY value.
 *
 * @exception SQLException
 *            if an error occurs while attempting to access the array.
 *
 * @exception SQLFeatureNotSupportedException
 *            if the JDBC driver does not support this method.
 */
Object getArray(int checkKind, String args);
{
}
```

### 块内注释

```c++
class Base
{
  /** The number of nodes, which is also used to determine the next node index. */
  protected int m_size = 0;

  /** The expanded names, one array element for each node. */
  protected SuballocatedIntVector m_exptype;

  /** First child values, one array element for each node. */
  protected SuballocatedIntVector m_firstch;

  /** Next sibling values, one array element for each node. */
  protected SuballocatedIntVector m_nextsib;
};

void func()
{
  /** if it is success */
  if (isSuccess())
  {
    ...
  }
  ...
}
```

### TODO 注释

```c++
synchronized public DTMIterator createDTMIterator(int node)
{
  /** @todo: implement this com.sun.org.apache.xml.internal.dtm.DTMManager abstract method */
  return null;
}
```

## 其它编程建议

### include 头文件的路径

项目内头文件应按照项目目录树结构排列，避免使用 UNIX 特殊的快捷目录 . (当前目录) 或 .. (上级目录)。

```
├─src
|   └─widget
|       └─my_widget.h
|       └─my_widget.cpp
|       └─...
├─translation
├─image
├─third_party
├─my_project.pro
```

我的编程习惯是把所有头文件和源文件统一放在 src 目录下（当然你也可以分开来放，头文件放在 include 目录，源文件放在 src 目录，这种做法一般是库采用的，工程应用程序不用考虑这些），那么包含头文件的时候写全路径，比如`#include "src/widget/my_widget.h"`。

### 避免隐式转换和比较

比如下面的代码，

```c++
int n = read(fd, buffer, sizeof(buffer));

if (n < sizeof(buffer))
    exit(EXIT_FAILURE);
else
    do_something();
```

假如 read 返回 -1，按照预期，代码应该执行 exit(EXIT_FAILURE) 退出程序，但因为 n 是 int 类型， sizeof 返回值是 unsigned int 的，两者比较时，int 类型的数会被自动转换为 unsigned int 类型。也就是说 int 类型的 -1 在进行比较时会被转换成一个非常大的 unsigned int 类型的数，具体数值大小是 (1 << 32) - 1。

### 智能指针代替原生指针

```c++
void something_often_to_do()
{
    int* p = new int[1024];
	
    ...
	
    if (func1() == false) /* 中途发生意外需要退出 */
    {
        ...
        return; /* 退出前却忘记释放内存 */
    }
	
    ...
	
    if (func2() == false) /* 中途退出的情况可能不止一种 */
    {
        ...
        return;
    }
	
    ...
	
    if (func3() == false) /* 中途退出的情况可能不止一种 */
    {
        ...
        return;
    }
	
    ...
	
    delete[] p;
}
```

大家一般都记得会在函数尾部释放内存，但往往会忘记在异常退出的地方加上 delete[] 语句。正如函数名所示，若这个函数需要被反复调用（比如几百 ms 调用一次），必然会出现内存泄漏。

最优雅的解决之法自然是自 C++ 11 引入的智能指针。

当然，你也可以使用 goto 语句，但 goto 这个东西，除非你知道自己在干什么，否则都不要用。

类似的问题很有很多，比如，这个函数是一个 thread-safe 函数，你在开头对它上锁，但却忘记在异常退出的时候给它解锁。

### 尽可能地避免产生内存碎片

<https://www.zhihu.com/question/51836333/answer/145693402> 很清楚地讲解了何谓内存碎片。

```c++
std::string name("Jack");
std::string word = std::string("I am") + name;
```

应该替换成下面这样，

```c++
std::string name("Jack");
std::string word;
word += "I am";
word += name;
```

当然用 append 代替 += 也可以。

当预知所需字符串长度，可以使用 reserve 指定，以防止扩容所带来的重新构造、复制等操作损耗。

### 尽量使用前置自增语句

前置自增与后置自增的区别主要是效率问题，参考下面的三条链接即知。

参考：

1. https://stackoverflow.com/questions/24886/is-there-a-performance-difference-between-i-and-i-in-c
2. https://stackoverflow.com/questions/4445706/post-increment-and-pre-increment-concept
3. https://stackoverflow.com/questions/24853/what-is-the-difference-between-i-and-i

### 尽可能地使用 assert 语句

参考：https://stackoverflow.com/questions/1571340/what-is-the-assert-function

个人觉得，意思就是 assert 等于 i make sure。

### 避免操作符的短路问题

考虑游戏玩家回血回蓝（魔法）刷新给客户端的逻辑。玩家每3秒回一点血，玩家每5秒回一点蓝，回蓝回血共用一个协议通知客户端，也就是说只要有回血或者回蓝就要把新的血量和魔法值通知客户端。

玩家的心跳函数`heartbeat()`在主逻辑线程被循环调用：

```c++
void  GamePlayer::Heartbeat()
{
    if (GenHP() || GenMP())
    {
        NotifyClientHPMP();
    }
}
```

如果 GenHP 回血了，就返回 true，否则 false；不一定每次调用 GenHP 都会回血，取决于是否达到 3 秒间隔。

如果 GenMP 回蓝了，就返回 true，否则 false；不一定每次调用 GenMP 都会回血，取决于是否达到 5 秒间隔。

实际运行发现回血回蓝逻辑不对，Word 麻，原来是操作符短路了，如果 GenHP() 返回 true 了，那 GenMP() 就不会被调用，就有可能失去回蓝的机会。OMG，你需要修改程序如下：

```c++
void GamePlayer::Heartbeat()
{
    bool hp = GenHP();
    bool mp = GenMP();
    if (hp || mp) 
    {   
        NotifyClientHPMP();
    }   
}
```
逻辑与（&&）跟逻辑或（||）有同样的问题， if (a && b) 如果 a 的表达式求值为 false，b 表达式也不会被计算。

有时候，我们会写出 if (ptr != nullptr && ptr->Do()) 这样的代码，这正是利用了操作符短路的语法特征。

摘自：<https://mp.weixin.qq.com/s?__biz=MjM5MjAwODM4MA==&mid=2650714384&idx=2&sn=a787e0f33a59162f2f121c5fe8b7be11&chksm=bea6c0c389d149d59f47769a0edb3059943e22a6105235a3c7a0039188dcbf33c14a5bb5f068&mpshare=1&scene=23&srcid=#rd>


### 尽可能使用无异常的函数

先说说起因吧。

最近写了个 sql 客户端对接 microsoft sql server，用的 ado，其中有个语句`select * from`，按道理返回的一定是个数字，所以我直接用的`std::stoi`转换，但因为用 ado 返回数据的时候转换错了，导致转换出的是乱数据（也就是非数字），进而导致`std::stoi`抛出异常。

`std::stoi`完全可以用`atoi`代替。

就像这次的不可思议，不作为比异常抛出更好。

### 不要用尤达条件式

尤达条件式含义见维基 [尤达条件式](https://zh.wikipedia.org/wiki/%E5%B0%A4%E9%81%94%E6%A2%9D%E4%BB%B6%E5%BC%8F)。

之前我是推荐使用尤达条件式的，因为有好几次都是写错了，明明是判断等于，结果少写了个 =，就变成了赋值。现在之所以不推荐了，主要是因为两点：

1. 可读性
2. 对于两个左值不再适用

## 参考

- [Google 开源项目风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/)
- [Stackoverflow. C/C++ include header file order](https://stackoverflow.com/questions/2762568/c-c-include-header-file-order)
