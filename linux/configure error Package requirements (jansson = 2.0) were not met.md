```shell
checking for JANSSON... no
configure: error: Package requirements (jansson >= 2.0) were not met:

No package 'jansson' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables JANSSON_CFLAGS
and JANSSON_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```

这个库在 make install 后生成一个 jansson.pc 文件，拿到它的路径，然后把下面的代码放进 /etc/profile 里，

```
export PKG_CONFIG_PATH=/home/hapoa/pkgconfig/
```

接着执行`source /etc/profile`，再重新执行`./configure .....`即可。
