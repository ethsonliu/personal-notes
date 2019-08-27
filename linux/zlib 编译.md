下载：<https://www.zlib.net/>

### Linux

解压进入目录，

```shell
./configure --prefix=$PWD/__build
make && make install
```

如果是交叉编译，先指定编译链，

```shell
export CC=/home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc
./configure --prefix=$PWD/__build
make && make install
```

