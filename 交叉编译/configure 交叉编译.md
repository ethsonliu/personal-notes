```shell
./configure --prefix=$PWD/build --host=arm-linux --enable-static CC=/home/work/project/cbox/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
```

--prefix 指定你要 make install 的目录。

--host 指定要编译的平台，一般是你的编译链前缀，这个例子的完整名称是`arm-linux-gnueabihf`，取前缀`arm-linux`。

--enable-static 表示要编译成静态库，具体可以执行`./configure -h`查看相关参数。

CC 指定编译器，如果是 c 库，请用 gcc。

