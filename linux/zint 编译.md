下载：<https://sourceforge.net/projects/zint/files/zint/>

其它准备，下载编译：

- zlib 1.2.11- https://github.com/Hapoa/personal-notes/blob/master/linux/zlib%20%E7%BC%96%E8%AF%91.md
- libpng 1.6.37 - https://github.com/Hapoa/personal-notes/blob/master/linux/libpng%20%E7%BC%96%E8%AF%91.md

解压后，打开`backned/png.c`，头文件加入`#include <zlib.h>`。

### Linux



打开`backend/Makefile`，修改：

```makefile
CC := /home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc
prefix := /home/hapoa/packages/a40i/zint-2.4.1/__build
```

其中若报错，

```shell
/home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc -g -DZINT_VERSION=\"2.4.2\" -shared -Wl,-soname,libzint.so -o libzint.so.2.3.2 -I/home/hapoa/crosschain/arm-linux-gnueabi/arm-linux-gnueabi/include common.o render.o png.o library.o ps.o large.o reedsol.o gs1.o svg.o code.o code128.o 2of5.o upcean.o telepen.o medical.o plessey.o rss.o code16k.o dmatrix.o pdf417.o qr.o maxicode.o composite.o aztec.o code49.o code1.o gridmtx.o postal.o auspost.o imail.o `libpng12-config --I_opts --L_opts --ldflags` -lz -lm
/bin/sh: 1: libpng12-config: not found
ln -s libzint.so.* libzint.so
make[1]:正在离开目录 `/home/hapoa/packages/a40i/zint-2.4.2/backend'
make -C frontend/
make[1]: 正在进入目录 `/home/hapoa/packages/a40i/zint-2.4.2/frontend'
/home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc -Wall -I/home/hapoa/crosschain/arm-linux-gnueabi/arm-linux-gnueabi/include -g -DZINT_VERSION=\"2.4.2\" -I../backend -L../backend main.c -o zint -lzint
/home/hapoa/crosschain/arm-linux-gnueabi/bin/../lib/gcc/arm-linux-gnueabi/5.3.1/../../../../arm-linux-gnueabi/bin/ld: warning: libz.so.1, needed by ../backend/libzint.so, not found (try using -rpath or -rpath-link)
../backend/libzint.so：对‘png_create_write_struct’未定义的引用
../backend/libzint.so：对‘png_get_error_ptr’未定义的引用
../backend/libzint.so：对‘png_create_info_struct’未定义的引用
../backend/libzint.so：对‘png_write_info’未定义的引用
../backend/libzint.so：对‘png_write_row’未定义的引用
../backend/libzint.so：对‘png_write_end’未定义的引用
../backend/libzint.so：对‘png_set_IHDR’未定义的引用
../backend/libzint.so：对‘png_set_packing’未定义的引用
../backend/libzint.so：对‘png_init_io’未定义的引用
../backend/libzint.so：对‘png_set_compression_level’未定义的引用
../backend/libzint.so：对‘png_destroy_write_struct’未定义的引用
collect2: error: ld returned 1 exit status
make[1]: *** [zint] 错误 1
make[1]:正在离开目录 `/home/hapoa/packages/a40i/zint-2.4.2/frontend'
make: *** [zint] 错误 2
```

到`backend`找`libzint*`文件，发现已经存在，手动拷贝或`make install`即可。