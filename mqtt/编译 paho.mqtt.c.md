开源地址：https://github.com/eclipse/paho.mqtt.c

需要 openssl 的依赖，它的编译安装参照：https://github.com/Hapoa/personal-notes/blob/master/openssl/openssl%20%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85.md

如果只是编译动态库的话，直接`make`和`make install`即可，如果想要指定编译目录，修改 makefile 文件， prefix 改为你想编译的目录，其中 CC 为指定的编译器，注意把 ? 去掉。（make install 可能会失败，不然自己手动拷贝文件吧）

如果编译静态库，就得用 cmake ，建一个 build 目录，cd 进去，然后 

```
cmake .. -DPAHO_BUILD_STATIC=TRUE -DCMAKE_C_COMPILER=/home/work/project/cbox/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc -DPAHO_WITH_SSL=true
```

其中 DPAHO_BUILD_STATIC 指定为静态库，DCMAKE_C_COMPILER 指定编译器，DPAHO_WITH_SSL 指定开启 SSL。

如果需要指定 openssl 库，打开 src 目录下 CMakeLists.txt ，搜索`OPENSSL_LIB`和`OPENSSLCRYPTO_LIB`，改为指定的路径即可，如下，

```
FIND_PATH(OPENSSL_INCLUDE_DIR openssl/ssl.h
        HINTS ${OPENSSL_SEARCH_PATH}/include)
FIND_LIBRARY(OPENSSL_LIB NAMES ssl libssl ssleay32
        HINTS /home/work/lib ${OPENSSL_SEARCH_LIB_PATH})
FIND_LIBRARY(OPENSSLCRYPTO_LIB NAMES crypto libcrypto libeay32
      	HINTS /home/work/lib ${OPENSSL_SEARCH_LIB_PATH})
```

接着 make，然后如果你想 make install 到指定路径，修改 cmake_install.cmake 文件，其中的，

```
IF(NOT DEFINED CMAKE_INSTALL_PREFIX)
  SET(CMAKE_INSTALL_PREFIX "/home/jalyn/package/paho.mqtt.c-1.3.0/build")
ENDIF(NOT DEFINED CMAKE_INSTALL_PREFIX)
```

改为你想要的路径，然后再执行`make install`即可。（注意，如果是要安装到 /usr/local 就用 sudo make install，因为需要权限，如果只是本目录就不要用 sudo）
