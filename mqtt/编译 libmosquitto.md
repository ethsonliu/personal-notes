## Linux

Ubuntu 14.04 + mosquitto 1.6.2

下载地址：https://github.com/eclipse/mosquitto

依赖：openssl 1.1.1，见 https://github.com/Hapoa/personal-notes/blob/master/linux/openssl%20%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85.md

```shell
sudo apt-get install xsltproc # 这应该是给 doc 用的，装不装无所谓
```

虽然也会报错，

```
warning: failed to load external entity "http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl"
compilation error: file manpage.xsl line 3 element import
xsl:import : unable to load http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl
compilation error: file libmosquitto.3.xml line 4 element refentry
xsltParseStylesheetProcess : document is not a stylesheet
Makefile:78: recipe for target 'libmosquitto.3' failed
make[1]: *** [libmosquitto.3] Error 5
make[1]: Leaving directory '/home/hapoa/packages/mosquitto-1.6.2/man'
Makefile:17: recipe for target 'docs' failed
make: *** [docs] Error 2
```

其实库已经编译好了，

接着`sudo make install`，你就可以在`/usr/local/lib`找到了。

## Windows

需要准备的东西：

1. cmake-gui
2. mosquitto 源码，https://github.com/eclipse/mosquitto （我当前编译的是 1.6.2）
3. openssl，http://slproweb.com/products/Win32OpenSSL.html （当前的版本需要 1.1.1 以上支持，别下载 Light 的）
4. POSIX threads for win32，mosquitto 的 threading 支持，https://sourceware.org/pthreads-win32/，下载pthreads-w32-2-9-1-release.zip。

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/008.png)

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/006.png)

打开 libmosquitto，右键，【属性】-【C/C++】-【常规】-【附加包含目录】-【编辑】进去，可以看到这样的一行，`C:\pthreads\Pre-built.2\include`，把下载的`POSIX threads for win32`放在这个路径下，注意大小写和标点。

ctrl + f5，编译，可能会报错：

```
C2011	“timespec”:“struct”类型重定义	libmosquitto	C:\pthreads\Pre-built.2\include\pthread.h
```

打开`POSIX threads for win32`里边的`pthread.h`，在顶部加入`#define HAVE_STRUCT_TIMESPEC`，重新ctrl+f5 即可（参考：https://stackoverflow.com/questions/33557506/timespec-redefinition-error）。

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/007.png)

参考：[Win7下编译mosquitto源码](https://blog.csdn.net/Netown_Ethereal/article/details/41981103)