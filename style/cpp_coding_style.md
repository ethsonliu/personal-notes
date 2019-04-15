1. 头文件`define`保护，项目名+完整路径+文件名
2. `#include`顺序，参照 https://stackoverflow.com/questions/2762568/c-c-include-header-file-order
3. 命名空间，与类名规则一致，比如`namespace WebSearch {  ... }`
4. 尽量使用`explicit`修饰类成员函数
5. 使用 C++ 的类型转换, 如 `static_cast<>()`. 不要使用 `int y = (int)x` 和 `int y = int(x)` 等转换方式
6. 尽可能以 `sizeof(varname)` 代替 `sizeof(type)`
7. 枚举命名`enum class ShapeName { eHorLine, eVerLine, eWidth };`
8. 函数名，`getName()`和`setName()`，一一对应
9. 注释风格遵循 Doxygen，类内、函数体内等注释使用`//`
10. 对于静态存储周期（全局、静态，包括类内静态和函数内静态）的变量，其风格如下，全局加前缀`g`，静态加前缀's'，例如`int gDaysInAWeek = 7`，`static int sDaysInAWeek = 7`。
11. 如果一个常量想多个单元引用的话，需要显式以 `extern` 声明，`extern const int gcDaysInAWeek = 7;`，因为定义于全局作用域的`const`自带`static`属性
12. 如果一个常量只是本单元使用，则直接`const int scDaysInAWeek = 7;` 或 `static const int scDaysInAWeek = 7;`也可。
13. 而对于字符串，则有三种形式，`static const char* const scItemName = "Wali"`，`static const char* sfItemName = "Wali"`，`static char* const slItemName = "Wali"`。对于后两者，能不用就不同，这种命名方式只是个人使用。