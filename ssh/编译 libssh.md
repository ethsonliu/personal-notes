官网：<https://www.libssh.org/>

### windows

先安装 openssl，到<http://slproweb.com/products/Win32OpenSSL.html>，别下载 light 版本的，安装即可。

然后下载 libssh 最新版本，新建 build 目录，打开 cmake-gui.exe，注意下图，

![](<https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/010.png>)

生成静态库那个，我实际在工程中使用，却总是报错，

```
main.obj:-1: error: LNK2019: 无法解析的外部符号 __imp_ssh_free，该符号在函数 main 中被引用
```

但是生成的动态库可以用。

建议勾选 ZLIB，用于传输压缩用的。zlib 的官网：<https://zlib.net/>，下载编译就可以，很简单的这个，然后这个我们也生成动态库。还有一点，你生成的目录`build`里会有一个`zconf.h`的文件，请把它复制到库的头文件一起，这也是 libssh 要用的。

如果编译 libssh 说找不到 ZLIB 的位置，那就手动指定，点击 cmake-gui.exe 上的 Advanced，就会出现 ZLIB_INCLUDE_DIR 和 ZLIB_LIBRARY_RELEASE，手动填写即可，

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/011.png)

