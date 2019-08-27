下载：<http://www.libpng.org/pub/png/libpng.html>

其余准备：先编译 zlib 库，参见：<https://github.com/Hapoa/personal-notes/blob/master/linux/zlib%20%E7%BC%96%E8%AF%91.md>

### Linux

交叉编译的话，

```shell
./configure --host=arm-linux CC=/home/hapoa/crosschain/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc --prefix=$PWD/__build
```





### 参考

- <https://blog.csdn.net/dreamInTheWorld/article/details/54933859>
- <https://blog.csdn.net/water_cow/article/details/8728154>