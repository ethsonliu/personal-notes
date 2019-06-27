```shell
./configure --prefix=$PWD/build --host=arm-linux --enable-static --enable-shared=no CC=/home/jalyn/crosschain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
```

--prefix 指定你要 make install 的目录。

--host 指定要编译的平台，一般是你的编译链前缀，这个例子的完整名称是`arm-linux-gnueabihf`，取前缀`arm-linux`。

--enable-static 表示要编译成静态库，具体可以执行`./configure -h`查看相关参数。

--enable-shared=no 表示不生成动态库。

CC 指定编译器，如果是 c 库，请用 gcc。

如果有一个新的库需要依赖之前交叉编译的库，就把依赖的库和头文件分别放在交叉编译工具链的 lib 和 include 目录下。

#### 静态库

如果你要编译的是静态库，记住在生成的 Makefile 里`CFLAGS = -g -O2 `后面多加个`-shared -fPIC`。

#### pkgconfig

make install 后会生成一个 pkgconfig 目录，里边的文件路径需要放在 /etc/profile 里，当然如果你的工具链 lib 目录里有 pkgconfig 目录，就放在一起，统一管理，具体参照：[https://github.com/Hapoa/personal-notes/blob/master/linux/configure%20error%20Package%20requirements%20(jansson%20%3D%202.0)%20were%20not%20met.md](https://github.com/Hapoa/personal-notes/blob/master/linux/configure%20error%20Package%20requirements%20(jansson%20%3D%202.0)%20were%20not%20met.md)

#### 没有 configure，只有 makefile

就需要修改 makefile 文件了，找到 prefix 指定要 make install 的目录，一般是，

```shell
ifndef prefix
	prefix = $(shell pwd)/build
endif
```

CC 指定编译器，例如，

```shell
CC = /home/work/project/cbox/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
```

