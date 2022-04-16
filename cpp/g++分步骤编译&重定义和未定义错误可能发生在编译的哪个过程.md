## 重定义

https://blog.csdn.net/LaoJiu_/article/details/68060382

## 未定义

发生在最后一步链接。

注意，未定义和未声明的区别。如果当前 cpp 文件没有声明一个函数，而你又用了，那就是报错未声明。

## g++ 分步骤编译

参考：https://m.haicoder.net/gcc/gcc-distributed-compile-cpp.html

**预处理**

通过给 g++ 指令添加 -E 选项，即可轻松实现令 GCC 编译器只对目标源程序进行预处理操作。比如：
```
[root@haicoder demo]# g++ -E demo.cpp -o demo.i
[root@haicoder demo]# ls
demo.cpp  demo.i
```
注意，由于编译阶段需要用到预处理的结果，因此这里必须使用 -o 选项将该结果输出到指定的 demo.i 文件中（ Linux 系统中，通常用 “.i” 或者 “.ii” 作为 C++ 程序预处理后所得文件的后缀名）。

**编译**

值得一提的是，编译阶段针对的将不再是 demo.cpp 源文件，而是 demo.i 预处理文件。对预处理文件进行编译操作，实际上就是对 demo.i 文件做进一步的语法分析，并生成对应的汇编代码文件（Linux 发行版通常以 “.s” 作为其后缀名）。

通过给 g++ 指令添加 -S 选项，即可令 GCC 编译器仅对指定预处理文件做编译操作。例如：
```
[root@haicoder demo]# g++ -S demo.i
[root@haicoder demo]# ls
demo.cpp  demo.i  demo.s
```
和预处理阶段不同，即便这里不使用 -o 选项，编译结果也会输出到和预处理文件同名（后缀名改为 .s）的新建文件中。

**汇编**

汇编阶段就是将之前生成的汇编代码文件（demo.s）做进一步转换，生成对应的机器指令。通过给 g++ 指令添加 -c 选项，即可令 GCC 编译器仅对指定的汇编代码文件做汇编操作。例如：
```
[root@haicoder demo]# g++ -c demo.s
[root@haicoder demo]# ls
demo.cpp  demo.i  demo.o  demo.s
```
显然，默认情况下汇编操作会自动生成一个和汇编代码文件名称相同、后缀名为 .o 的二进制文件（又称为目标文件）。

**链接**

目标文件已经是二进制文件，与可执行文件的组织形式类似，只是有些函数和全局变量的地址还未找到，因此还无法执行。链接的作用就是找到这些目标地址，将所有的目标文件组织成一个可以执行的二进制文件。

完成链接操作，并不需要给 g++ 添加任何选项，只要将汇编阶段得到的 demo.o 作为参数传递给它，g++ 就会在其基础上完成链接操作。例如：
```
[root@haicoder demo]# g++ demo.o
[root@haicoder demo]# ls
a.out  demo.cpp  demo.i  demo.o  demo.s
```

在链接阶段，如果不使用 -o 选项将执行结果输出到指定文件，则 g++ 会默认创建一个名为 a.out 的可执行文件，并将执行结果输出到该文件中。经过以上 4 步，最终生成了 a.out 可执行文件，我们可以尝试运行该文件，查看其结果是否正确：

```
[root@haicoder demo]# ./a.out
```

显然，该结果和我们的预期相符。除此之外，如果读者不想执行这么多条指令，但想获得预处理、编译、汇编以及链接这 4 个过程产生的中间文件，可以执行如下指令：
```
[root@haicoder demo]# g++ demo.cpp -save-temps
[root@haicoder demo]# ls
a.out  demo.c  demo.cpp  demo.ii  demo.o  demo.s
```
可以看到，通过给 g++ 添加 -save-temps 选项，可以使 GCC 编译器保留编译源文件过程中产生的所有中间文件。
