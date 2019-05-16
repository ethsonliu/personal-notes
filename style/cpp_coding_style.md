1. 头文件`define`保护，项目名+完整路径+文件名

2. `#include`顺序，参照 https://stackoverflow.com/questions/2762568/c-c-include-header-file-order

3. 命名空间，与类名规则一致，比如`namespace WebSearch {  ... }`

4. 尽量使用`explicit`修饰类成员函数

5. 使用 C++ 的类型转换, 如 `static_cast<>()`. 不要使用 `int y = (int)x` 和 `int y = int(x)` 等转换方式

6. 尽可能以 `sizeof(varname)` 代替 `sizeof(type)`

7. 枚举命名`enum class ShapeName { eHorLine, eVerLine, eWidth };`

8. 函数名，`getName()`和`setName()`，一一对应

9. 注释风格遵循 Doxygen，类内、函数体内等注释使用`/*    */`

10. 对于静态存储周期（全局、静态，包括类内静态和函数内静态）的变量，其风格如下，全局加前缀`g`，静态加前缀's'，例如`int gDaysInAWeek = 7`，`static int sDaysInAWeek = 7`。

11. 常量字母都是大写，不管其存储周期为何

12. 如果一个常量想多个单元引用的话，需要显式以 `extern` 声明，`extern const int DAYS_IN_A_WEEK = 7;`，因为定义于全局作用域的常量自带`static`属性，也就是说一个常量是可以定义于头文件的

13. 关于程序的版本（用字符串或者数字表示），放在一个`version.h`中定义，该文件只放版本的宏，宏的名称为`$appname_VERSION`。

    ```c++
    const char* const APPNAME_VERSION = "1.6";
    ```


14. 考虑到日志，和一些几乎每个文件都需要的宏，或者头文件，可以统一放在`global.h`，比如，

   ```
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
   
extern float gRatio;
#define FIT(n) static_cast<int>(((n) * gRatio))
   ```

   