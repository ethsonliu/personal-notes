下载：<https://sourceforge.net/projects/libpng/files/>

其余准备：先编译 zlib 库，当前版本 1.2.11，参见：<https://github.com/Hapoa/personal-notes/blob/master/linux/zlib%20%E7%BC%96%E8%AF%91.md>

当前 libpng 版本为 1.6.37。

### Linux

交叉编译的话，

```shell
./configure --host=arm-linux CC=/home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc --prefix=$PWD/__build

make && make install
```





### 参考

- <https://blog.csdn.net/dreamInTheWorld/article/details/54933859>
- <https://blog.csdn.net/water_cow/article/details/8728154>