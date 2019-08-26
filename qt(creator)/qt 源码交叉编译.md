## 准备工作

- Qt 源码：<https://download.qt.io/archive/qt/5.10/5.10.1/single/qt-everywhere-src-5.10.1.tar.xz>

- 编译平台：Ubuntu 14.04 x64

- 交叉编译工具链：arm-linux-gnueabi

- 若在虚拟机上，请确保系统内存足够，我在虚拟机上加到了 3G。

- 检查编译的工具，如果没有就安装，

  ```shell
  sudo apt-get install autoconf
  sudo apt-get install automake
  sudo apt-get install libtool
  ```

## 进行编译

解压 Qt 源码，编辑`/qtbase/mkspecs/linux-arm-gnueabi-g++/qmake.conf`，

```
#
# qmake configuration for building with arm-linux-gnueabi-g++
#

MAKEFILE_GENERATOR      = UNIX
CONFIG                 += incremental
QMAKE_INCREMENTAL_STYLE = sublib

include(../common/linux.conf)
include(../common/gcc-base-unix.conf)
include(../common/g++-unix.conf)

# modifications to g++.conf
QMAKE_CC                = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc
QMAKE_CXX               = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-g++
QMAKE_LINK              = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-g++
QMAKE_LINK_SHLIB        = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-g++

# modifications to linux.conf
QMAKE_AR                = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-ar cqs
QMAKE_OBJCOPY           = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-objcopy
QMAKE_NM                = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-nm -P
QMAKE_STRIP             = /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-strip
load(qt_config)
```

将编译链改为你要使用的编译链路径。

进入源码根目录，即`configure`所在的目录，执行以下命令，

```shell
./configure -prefix /home/hapoa/crosschain/qt-everywhere-src-5.10.1-a40i/__build -release -opensource -make libs -xplatform linux-arm-gnueabi-g++ -optimized-qmake -pch -qt-libjpeg -qt-zlib -no-opengl -skip qt3d -skip qtcanvas3d -skip qtpurchasing -no-sse2 -no-openssl -no-cups -no-glib -no-iconv -nomake examples -nomake tools -skip qtvirtualkeyboard
```

上面的准备工作里有内存的要求，就在这里，如果内存不够，会报错：

```
Creating qmake... ........virtual memory exhausted: 无法分配内存

或者

Creating qmake... .g++: internal compiler error: 已杀死 (program cc1plus)
```

等待几分钟，若成功，继续执行`make`编译库，需要等待 1 个小时以上，

成功后，`make install`就会安装到`/home/hapoa/crosschain/qt-everywhere-src-5.10.1-a40i/__build`。

## 参考

- <<https://blog.csdn.net/u014213012/article/details/87859263>>