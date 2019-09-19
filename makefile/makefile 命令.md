## 目录

- [LFLAGS](#LFLAGS)
- [sysroot](#sysroot)

## LFLAGS

指定 Lex 文法分析器参数。

## sysroot

指定头文件和库所在的根目录。例如，如果头文件和库通常分别位于`/usr/include`和`/usr/lib`中，则`--sysroot=/mydir`将导致编译器在`/mydir/usr/include`和`/mydir/usr/lib`中搜索头文件和库。对于嵌入式开发，我们可以使用`--sysroot`选项指定头文件和库的根目录。

参考：

 - [Improved sysroot support in Intel C++ Compiler for cross compile](https://software.intel.com/en-us/articles/improved-sysroot-support-in-intel-c-compiler-for-cross-compile)
 - [Building and cross-compile tutorial](http://www.fabriziodini.eu/posts/cross_compile_tutorial/)
 - [Cross compilation: GCC ignores --sysroot](https://stackoverflow.com/questions/17603213/cross-compilation-gcc-ignores-sysroot)
