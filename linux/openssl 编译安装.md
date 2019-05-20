OpenSSL库的安装

官网：http://www.openssl.org

下载页面：http://www.openssl.org/source/

选择最新版本下载

<http://www.openssl.org/source/openssl-1.0.1s.tar.gz>

解压：

`tar –zxvf openssl-1.0.1s.tar.gz，解压目录为：openssl-1.0.1s`

然后进入到 `cd openssl-1.0.1s`，进行配置、编译、安装

配置

`./config`

编译

`make`

安装

`sudo make install`，安装在`/usr/local/`和`/usr/local/bin`和`/usr/local/include`

参考：https://skynineblog.wordpress.com/2016/04/08/linux%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85-openssl%E4%B8%A4%E7%A7%8D%E6%96%B9%E6%B3%95/
