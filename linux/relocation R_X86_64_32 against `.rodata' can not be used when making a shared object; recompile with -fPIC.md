这个问题怎么遇到的呢？

有一个项目需要使用 paho.mqtt.c，我把它编译成静态了，然后项目中链接使用，在链接的时候会同时生成一个动态库供其它功能使用，报错就在静态库被链接到动态库上时出错了。

解决的办法很简单，在你编译静态库的时候，`CFLAGS`或者`CXXFLAGS`多加个`-fPIC`，这样在编译哪些`*.c`和`*.cpp`是时候会添加`-fPIC`，等价于下面这样：

```shell
gcc -o static_shared.o -fPIC -c static.c
gcc -o dynamic.o -fPIC -c dynamic.c
```

注意，不管这个静态库，连同它所依赖的库都要加`-fPIC`。

然后生成动态库的，`CFLAGS`或者`CXXFLAGS`加`-shared`即可。