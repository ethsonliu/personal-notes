Qt 5.9 交叉编译的时候是带有 webengine的，然后移到我的电脑上编译一个 qt 程序，报错

```
/home/hapoa/crosschain/arm-linux-gnueabi/bin/ld: 找不到 crt1.o: 没有那个文件或目录
/home/hapoa/crosschain/arm-linux-gnueabi/bin/ld: 找不到 crti.o: 没有那个文件或目录
/home/hapoa/crosschain/arm-linux-gnueabi/bin/ld: 找不到 -lpthread
/home/hapoa/crosschain/arm-linux-gnueabi/bin/ld: 找不到 -lm
```

发现在交叉编译 qt 的时候，有一个 sysroot 目录，里边就有上面的四个文件，然后拷贝到我机器上的交叉编译链里，当然，不能一个一个的拷贝到原先交叉编译连里的 lib 和 include，太多了，一定有指令让它可以找到 sysroot 下的东西，折腾了一会儿，解决了。

在 qt 程序 pro 文件加入，

```
QMAKE_LFLAGS += --sysroot=/home/hapoa/crosschain/arm-linux-gnueabi/arm-linux-gnueabi/sysroot
```

它就找到了。

当时，也出现一个报错，找不到 features.h，这个头文件就在这个 sysroot 里，因此又多加了一句，

```
INCLUDEPATH += /home/hapoa/crosschain/arm-linux-gnueabi/arm-linux-gnueabi/sysroot/usr/include
```

