

官网：http://www.openssl.org

下载页面：http://www.openssl.org/source/

解压：

`tar –zxvf openssl-1.0.1s.tar.gz，解压目录为：openssl-1.0.1s`

然后进入到 `cd openssl-1.0.1s`，进行配置、编译、安装。

### ubuntu

配置`./Configure --prefix=$PWD/__build `，如果要编译 32 位的，则`./Configure --prefix=$PWD/__build -m32 linux-generic32`，

编译`make`，安装`sudo make install`，安装在`/usr/local/`和`/usr/local/bin`和`/usr/local/include`。

参考：

- <https://skynineblog.wordpress.com/2016/04/08/linux%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85-openssl%E4%B8%A4%E7%A7%8D%E6%96%B9%E6%B3%95/>
- <https://stackoverflow.com/questions/7835596/how-do-i-compile-openssl-in-32-bit-mode-on-a-64bit-system>

### 嵌入式

配置`./Configure --prefix=$PWD/__build no-asm linux-armv4`，

其中`linux-armv4`是要编译的平台，可以使用`./Configure -h`查看所有可支持的平台。

生成 makefile 后，进去修改`CROSS_COMPILE`改为你的编译器，如果需要`CFLAGS`加`-fPIC`。

```
CROSS_COMPILE=/home/hapoa/crosschain/am335x/bin/arm-arago-linux-gnueabi-
CC=$(CROSS_COMPILE)gcc
CXX=$(CROSS_COMPILE)g++
CPPFLAGS=
CFLAGS=-Wall -O3
CXXFLAGS=-Wall -O3
LDFLAGS= 
EX_LIBS= 
```