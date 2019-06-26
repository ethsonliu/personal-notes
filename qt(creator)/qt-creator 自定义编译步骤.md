qt-creator可以自动编译步骤，偶然间发现的，方便好多。

我有一个工程，对应三个平台，也就需要三个编译工具链，也就对应三个 Makefile。

每次编译一个平台的时候，就要把对应的 Makefile 拷贝（copy）到当前工程根目录下，然后执行 make。现在我想快速地切换各个编译平台怎么办呢？

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/009.png)

