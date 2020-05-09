## 准备工作

1. 下载 sqlite，前往 [Sqlite Github Download](https://github.com/sqlite/sqlite/releases)
2. 交叉编译链

## 交叉编译

```
./configure --prefix=$PWD/build --host=arm-linux --enable-static --enable-shared CC=/home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc

# 如果上一步没报错，就会出现 Makefile 文件，接着
make && make install
```

可以参考：[configure 交叉编译](https://github.com/EthsonLiu/personal-notes/blob/master/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91/configure%20%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91.md)
