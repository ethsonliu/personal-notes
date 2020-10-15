官网：<https://iperf.fr/>

## 服务端

```shell
yum -y install gcc make
cd /tmp
wget https://iperf.fr/download/source/iperf-3.1.3-source.tar.gz
tar zxvf iperf-3.1.3-source.tar.gz
cd iperf-3.1.3
./configure
make
make install
```

打开 `/etc/profile`，加入以下命令使其开机启动，

```shell
/usr/local/bin/iperf3 -s -D -p 5201
```

接着执行 `nohup /usr/local/bin/iperf3 -s -D -p 5201 >/dev/null 2>&1 &` 来启动。

这样，服务端就开始监听 5201 端口了。

