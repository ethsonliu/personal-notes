## Linux & Arm

我是在 Ubuntu 14.04 上编译的，当前版本 mosquitto 1.6.2，下载请到这里 <https://github.com/eclipse/mosquitto>。

如果要使用 SSL，得先编译依赖 openssl 1.1.1，编译方法见 [openssl 编译安装](https://github.com/Hapoa/personal-notes/blob/master/openssl/openssl%20%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85.md)。

mosquitto 编译同时也会编译 doc，但这个可要可不要。如果一定需要，你可以通过以下命令安装依赖，

```shell
sudo apt-get install xsltproc
```

如果不需要，则可以进入 `config.mk`，修改它跳过编译 doc，

```
# Build man page documentation by default.
WITH_DOCS:=no
```

如果不安装这个依赖或者也不修改 `WITH_DOCS:=no`，编译最后可能会报错，

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

若想指定安装目录，打开 `config.mk` 修改 `prefix`。

接着 `sudo make install`，你就可以在你指定的目录或默认 `/usr/local/lib` 找到可执行程序、头文件和动态库。

如果要编译 32 位的（当前机器 Ubuntu 14.04 x64），则指定，

```
make CCFLAGS="-m32 " CPPFLAGS="-m32 " CXXFLAGS="-m32" CFLAGS="-m32" LDFLAGS="-m32"
```

如果是交叉编译的话，还得修改 `config.mk` 里的 `STRIP` 为，

```
STRIP?=/home/hapoa/crosschain/am335x/bin/arm-arago-linux-gnueabi-strip
```

然后，

```
make CC=/home/hapoa/crosschain/am335x/bin/arm-arago-linux-gnueabi-gcc CXX=/home/hapoa/crosschain/am335x/bin/arm-arago-linux-gnueabi-g++
```

如果遇到错误：

```shell
cc -L../lib pub_client.o pub_shared.o client_shared.o client_props.o -o mosquitto_pub ../lib/libmosquitto.so.1
../lib/libmosquitto.so.1: undefined reference to OPENSSL_sk_num' ../lib/libmosquitto.so.1: undefined reference to SSL_CTX_up_ref'
../lib/libmosquitto.so.1: undefined reference to OPENSSL_sk_pop_free' ../lib/libmosquitto.so.1: undefined reference to SSL_CTX_set_alpn_protos'
../lib/libmosquitto.so.1: undefined reference to OPENSSL_sk_value' ../lib/libmosquitto.so.1: undefined reference to OPENSSL_init_crypto'
../lib/libmosquitto.so.1: undefined reference to SSL_CTX_set_options' ../lib/libmosquitto.so.1: undefined reference to TLS_client_method'
../lib/libmosquitto.so.1: undefined reference to `ASN1_STRING_get0_data'
```

在 `config.mk` 中加入一行，

```
ifeq ($(WITH_TLS),yes)
	BROKER_LDADD:=$(BROKER_LDADD) -lssl -lcrypto
	LIB_LIBADD:=$(LIB_LIBADD) -lssl -lcrypto
	BROKER_CPPFLAGS:=$(BROKER_CPPFLAGS) -DWITH_TLS
	LIB_CPPFLAGS:=$(LIB_CPPFLAGS) -DWITH_TLS
	PASSWD_LDADD:=$(PASSWD_LDADD) -lcrypto
	CLIENT_CPPFLAGS:=$(CLIENT_CPPFLAGS) -DWITH_TLS
	STATIC_LIB_DEPS:=$(STATIC_LIB_DEPS) -lssl -lcrypto
  
  # 加入这一行
  CLIENT_LDADD:=$(CLIENT_LDADD) -lssl -lcrypto

	ifeq ($(WITH_TLS_PSK),yes)
		BROKER_CPPFLAGS:=$(BROKER_CPPFLAGS) -DWITH_TLS_PSK
		LIB_CPPFLAGS:=$(LIB_CPPFLAGS) -DWITH_TLS_PSK
		CLIENT_CPPFLAGS:=$(CLIENT_CPPFLAGS) -DWITH_TLS_PSK
	endif
endif
```

## Windows

需要准备的东西：

1. cmake-gui
2. mosquitto 源码，https://github.com/eclipse/mosquitto （我当前编译的是 1.6.2）
3. openssl，http://slproweb.com/products/Win32OpenSSL.html （当前的版本需要 1.1.1 以上支持，别下载 Light 版的）
4. POSIX threads for win32，mosquitto 的 threading 支持，<https://sourceware.org/pthreads-win32/>，下载 pthreads-w32-2-9-1-release.zip。

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/008.png)

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/006.png)

打开 libmosquitto，右键，【属性】-【C/C++】-【常规】-【附加包含目录】-【编辑】进去，可以看到这样的一行，`C:\pthreads\Pre-built.2\include`，把下载的 pthreads-win32 放在这个路径下（它默认就是在这个目录的，下面还要链接库，省事点，按它默认的来比较方便），注意大小写和标点。

ctrl + f5，编译，可能会报错：

```
C2011	“timespec”:“struct”类型重定义	libmosquitto	C:\pthreads\Pre-built.2\include\pthread.h
```

打开 pthreads-win32 里边的`pthread.h`，在顶部加入`#define HAVE_STRUCT_TIMESPEC`，重新 ctrl+f5 即可（此处参考 <https://stackoverflow.com/questions/33557506/timespec-redefinition-error>）。

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/007.png)

参考：[Win7下编译mosquitto源码](https://blog.csdn.net/Netown_Ethereal/article/details/41981103)
