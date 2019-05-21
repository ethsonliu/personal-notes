1. 头文件`define`保护，项目名+完整路径+文件名

2. `#include`顺序，参照 https://stackoverflow.com/questions/2762568/c-c-include-header-file-order

3. 命名空间，与类名规则一致，比如`namespace WebSearch {  ... }`

4. 尽量使用`explicit`修饰类成员函数

5. 使用 C++ 的类型转换, 如 `static_cast<>()`. 不要使用 `int y = (int)x` 和 `int y = int(x)` 等转换方式

6. 尽可能以 `sizeof(varname)` 代替 `sizeof(type)`

7. 枚举命名`enum class ShapeName { HorLine, VerLine, Width };`

8. 函数名，`getName()`和`setName()`，一一对应，set和get的内容应该和你要修改的变量名字一致，函数名长点没事

9. 注释风格遵循 Doxygen，类内、函数体内等注释使用`/*    */`

```c++
/**
 * @fn modify_value
 *
 * This is a function.
 *
 * @param[in]  val      Value calculations are based off.
 * @param[out] variable Function output is written to this variable.
 *
 * @return 对返回值的描述，或者用下面的
 * @retval 0 失败
 * @retval 1 成功
 *
 * @exception 对异常的描述
 * @note       注解
 * @attention  注意
 */
int modify_value(int val, int *variable)
{
    val *= 5;
    int working = val % 44;
    *variable = working;
    return 1;
}
```

10. 对于静态存储周期（全局、静态，包括类内静态和函数内静态）的变量，全局加前缀`g`，例如`int gDaysInAWeek = 7`，静态变量和普通变量一样，`static int daysInAWeek = 7`。注意，面向对象编程中，你几乎用不到全局变量，请考虑把它改为类内静态变量，例如`int MYClass::daysInAWeek = 7;`

11. 常量都大写，不管其存储周期为何，`const int DAYS_IN_A_WEEK = 7;`

12. 如果一个常量想多个单元引用的话，需要显式以 `extern` 声明，`extern const int DAYS_IN_A_WEEK = 7;`，因为定义于全局作用域的常量自带`static`属性，也就是说一个常量是可以定义于头文件的

13. 关于程序的版本（用字符串或者数字表示），放在一个`version.h`中定义，该文件只放版本的宏，宏的名称为`$appname_VERSION`

```c++
const char* const APPNAME_VERSION = "1.6";
```

14. 考虑到日志，和一些几乎每个文件都需要的宏，或者头文件，可以统一放在`global.h`，比如，

```c++
#ifndef HC_SRC_GLOBAL_H
#define HC_SRC_GLOBAL_H

#ifdef DESKTOP_APPLICATION
#include <QDebug>
#include <QString>
#endif
   
#include <plog/Log.h> // for log
#include <cassert> // for function assert
   
#define UNUSED(e) (void)e
   
#endif /* HC_SRC_GLOBAL_H */
```

15. 文件结构最好将它们从其他正常`h/cpp`分离开，和`main.cpp`放在一起，即大致如下，

```
├─src
│  └─folder1
│  └─folder1
│  └─folder1
│  └─folder1
│  main.cpp
│  version.h
|  global.h
|  global.cpp # 如果你的程序需要自定义一些类如标准 algorithm 内的函数，可以放在这里，比如字符串割
|  change_log.md
```

16. 程序都是有发布说明的，命名为`change_log.md`

17. 桌面应用程序因为牵扯到屏幕适配，目前的做法是在运行前获取屏幕大小，并得到比例，将这个比例放在`global.h`中，例如，

```
// global.h

#include "src/resolution.h" /* 屏幕适配方案 */
#define FIT(n) Resolution::fit(n) /* fit 函数实现了适配比与 n 的乘积，并返回一个 int */
```

18. 若是 Qt 程序，接口全用 Qt 自己提供的，以保证兼容性。若不是 Qt 程序，可以使用 CMake，需要注意`assert`的使用，尤其是在`release`模式下，对宏`NDEBUG`的定义