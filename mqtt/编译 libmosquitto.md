Ubuntu 14.04 + mosquitto 1.6.2

下载地址：https://github.com/eclipse/mosquitto

依赖：openssl 1.1.1，见 https://github.com/Hapoa/personal-notes/blob/master/linux/openssl%20%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85.md

```shell
sudo apt-get install xsltproc # 这应该是给 doc 用的，装不装无所谓
```

虽然也会报错，

```
warning: failed to load external entity "http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl"
compilation error: file manpage.xsl line 3 element import
xsl:import : unable to load http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl
compilation error: file libmosquitto.3.xml line 4 element refentry
xsltParseStylesheetProcess : document is not a stylesheet
Makefile:78: recipe for target 'libmosquitto.3' failed
make[1]: *** [libmosquitto.3] Error 5
make[1]: Leaving directory '/home/hapoa/packages/mosquitto-1.6.2/man'
Makefile:17: recipe for target 'docs' failed
make: *** [docs] Error 2
```

其实库已经编译好了，

接着`sudo make install`，你就可以在`/usr/local/lib`找到了。