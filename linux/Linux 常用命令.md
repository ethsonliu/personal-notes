## 目录

- [vi](#vi)
  - [删除当前行](#删除当前行)
  - [移动到当前行首行尾](#移动到当前行首行尾)
  - [撤销与恢复撤销](#撤销与恢复撤销)
  - [插入空白行](#插入空白行)
- [查找](#查找)
  - [可执行文件](#可执行文件)
  - [进程](#进程)
  - [已安装程序](#已安装程序)
  - [普通文件](#普通文件)
- [查看某个进程的使用情况](#查看某个进程的使用情况)
- [查看某个进程启动的个数](#查看某个进程启动的个数)
- [端口占用](#端口占用)
- [查看磁盘和文件大小](#查看磁盘和文件大小)
- [防火墙](#防火墙)
  - [firewall](#firewall)
  - [iptables](#iptables)
- [IO CPU 内存 网络](#IO-CPU-内存-网络)
  - [IO](#IO)
  - [CPU](#CPU)
  - [内存](#内存)
  - [网络](#网络)

## vi

### 删除当前行

```vi
dd
```

### 移动到当前行首行尾

```vi
0 ## 行首，注意是键盘上那一行数字中的 0
$ ## 收尾
```

### 撤销与恢复撤销

```vi
u
ctrl + r
```

### 插入空白行

```
o  ## 在光标所在位置的，下一行打开新行，并进入 INSERT 模式
O  ## 在光标所在位置的，上一行打开新行，并进入 INSERT 模式
```

## 查找

### 可执行文件

which 命令查找 PATH 变量指定的路径中，搜索某个系统命令的位置，默认返回第一个搜索结果，例如，

```shell
[root@~]# which openssl
/usr/bin/openssl
[root@~]# which make
/usr/bin/make
[root@~]# which make -a ## 返回所有结果，可用 man which 查看语法
/usr/bin/make
```

### 进程

是指查找当前运行的程序。

```shell
## 显示所有进程，带执行它的命令行
[root@EMQ /]# ps -ef
root      2612     1  0 Sep12 ttyS0    00:00:00 /sbin/agetty --keep-baud 115200 38400 9600 ttyS0 vt220
root      2613     1  0 Sep12 tty1     00:00:00 /sbin/agetty --noclear tty1 linux
root      2820     1  0 Sep12 ?        00:00:00 /sbin/dhclient -1 -q -lf /var/lib/dhclient/dhclient--eth0.leas
root      2900     1  0 Sep12 ?        00:00:51 /usr/sbin/rsyslogd -n
root      2907     1  0 Sep12 ?        00:02:06 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
root      2966     1  0 Sep12 ?        00:06:55 /usr/local/aegis/aegis_update/AliYunDunUpdate
root      3089     1  0 Sep12 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx     3090  3089  0 Sep12 ?        00:03:38 nginx: worker process
root      3392     1  0 Sep12 ?        00:00:00 /usr/sbin/sshd -D
root      3501     1  0 Sep12 ?        00:07:35 /usr/sbin/aliyun-service
root      3905     1  1 Sep12 ?        05:32:12 /usr/local/aegis/aegis_client/aegis_10_73/AliYunDun
root      4029     1  0 Sep12 ?        01:14:12 ./frps -c ./frps.ini
mysql     7402     1  0 Sep21 ?        00:02:17 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld
...
```
```shell
## 按名字查找 ssh
[root@EMQ /]# ps -ef|grep ssh
root      3392     1  0 Sep12 ?        00:00:00 /usr/sbin/sshd -D
root      8218 18317  0 20:07 pts/0    00:00:00 grep --color=auto ssh
root     18315  3392  0 Sep20 ?        00:00:01 sshd: root@pts/0
```

### 已安装程序

whereis 命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man 说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

和 find 相比，whereis 查找的速度非常快，这是因为 linux 系统会将 系统内的所有文件都记录在一个数据库文件中，当使用 whereis 和 locate 时，会从数据库中查找数据，而不是像 find 命令那样，通过遍历硬盘来查找，效率自然会很高。 

但是该数据库文件并不是实时更新，默认情况下时一星期更新一次，因此，我们在用 whereis 和 locate 查找文件时，有时会找到已经被删除的数据，或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。 

具体使用参考：<https://www.runoob.com/linux/linux-comm-whereis.html>

### 普通文件

find 有点暴力，遍历磁盘的查找。

```shell
find / -name openssl*
```

## 查看某个进程的使用情况

```shell
## 查看所有进程（同时显示执行时的命令）
top -c
```

```shell
## 显示进程 CPU 占比、内存占比、内存大小
[root@EMQ /]# ps -aux | grep frps
root      4029  0.4  0.6 114008 24124 ?        Sl   Sep12  74:11 ./frps -c ./frps.ini
root      8081  0.0  0.0 112708   980 pts/0    R+   20:05   0:00 grep --color=auto frps
```
其中 【0.0=CPU %，【1.1】=内存 %，【11984】=内存KB

```shell
## 查看进程详细状态
[root@EMQ /]# cat /proc/2913/status
Name:	frps ## 进程名
Umask:	0022
State:	S (sleeping)
Tgid:	4029
Ngid:	0
Pid:	4029 ## 进程 pid
PPid:	1 ## 父进程 pid
TracerPid:	0
Uid:	0	0	0	0
Gid:	0	0	0	0
FDSize:	256 ## 文件描述符的最大个数，具体参见下面的链接
Groups:	0 
VmPeak:	  114008 kB
VmSize:	  114008 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	   29040 kB
VmRSS:	   24428 kB ## 物理内存的大小
RssAnon:	   17268 kB
RssFile:	    7160 kB
RssShmem:	       0 kB
VmData:	  102996 kB
VmStk:	     132 kB
VmExe:	    4268 kB
VmLib:	       0 kB
VmPTE:	      88 kB
VmSwap:	       0 kB
Threads:	5
SigQ:	1/15076
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000001
SigCgt:	fffffffe7fc1fefe
CapInh:	0000000000000000
CapPrm:	0000001fffffffff
CapEff:	0000001fffffffff
CapBnd:	0000001fffffffff
CapAmb:	0000000000000000
Seccomp:	0
Speculation_Store_Bypass:	vulnerable
Cpus_allowed:	1
Cpus_allowed_list:	0
Mems_allowed:	00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:	0
voluntary_ctxt_switches:	28791781
nonvoluntary_ctxt_switches:	89475
```
详情参见：<https://www.cnblogs.com/youxin/p/5976194.html>

# 查看某个进程启动的个数

```shell
[root@iZ94xyihsxsZ home]# ps -ef|grep php-fpm|grep -v "grep"|wc -l
6
```

## 端口占用

显示所有 tcp，udp 的端口和进程等相关情况，
```shell
[root@EMQ /]# netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      943/php-fpm: master
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      944/redis-server 12
tcp        0      0 0.0.0.0:4431            0.0.0.0:*               LISTEN      14165/nginx: master
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      14165/nginx: master
tcp        0      0 0.0.0.0:2230            0.0.0.0:*               LISTEN      942/sshd
tcp6       0      0 :::4443                 :::*                    LISTEN      8772/ngrokd
tcp6       0      0 :::443                  :::*                    LISTEN      8772/ngrokd
tcp6       0      0 :::8000                 :::*                    LISTEN      8772/ngrokd
tcp6       0      0 :::3306                 :::*                    LISTEN      1235/mysqld
tcp6       0      0 :::61451                :::*                    LISTEN      859/dotnet
tcp6       0      0 :::8085                 :::*                    LISTEN      5164/node /var/loca
udp        0      0 120.76.211.89:123       0.0.0.0:*                           425/ntpd
udp        0      0 10.25.139.78:123        0.0.0.0:*                           425/ntpd
udp        0      0 127.0.0.1:123           0.0.0.0:*                           425/ntpd
udp        0      0 0.0.0.0:123             0.0.0.0:*                           425/ntpd
udp6       0      0 :::123                  :::*                                425/ntpd
```

## 查看磁盘和文件大小

```shell
## 查看磁盘整体使用情况
[root@EMQ /]# df -hl
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G  6.4G   31G  17% /
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  464K  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
tmpfs           379M     0  379M   0% /run/user/0
```

```shell
## 一个目录的大小
[root@EMQ /]# du -sh /usr/local/
292M	/usr/local/
[root@EMQ /]# du -sh /usr/
2.4G	/usr/
```

```shell
## 该路径下所有第一层的文件夹的大小（-d1）
[root@EMQ /]# du -h -d1 /usr/local/
8.3M	/usr/local/openresty
38M	/usr/local/share
48K	/usr/local/include
4.0K	/usr/local/libexec
2.2M	/usr/local/bin
4.0K	/usr/local/etc
4.0K	/usr/local/games
4.0K	/usr/local/sbin
1.5M	/usr/local/lib
4.0K	/usr/local/lib64
4.0K	/usr/local/src
231M	/usr/local/aegis
292M	/usr/local/
```



```shell
[root@EMQ home]# ls -l -AF ## 先查看这个目录
total 52100
-rwxrwxrwx 1 root    root        2013 Jun 14 13:28 au.sh*
-rwxrwxrwx 1 root    root       68689 Aug 24 10:04 certbot-auto*
drwxr-xr-x 2 root    root        4096 Sep 17 18:36 certs/
-rwxrwxrwx 1 root    root     3125224 Jul 31 15:42 data_interconnection-x86_64.AppImage*
-rwxrwxrwx 1 root    root    18692288 Jun 11 09:44 emqx-centos7-v3.0-beta.3.x86_64.rpm*
-rwxrwxrwx 1 root    root    20242888 Jun 10 16:44 emqx-centos7-v3.2-beta.2.x86_64.rpm*
-rwxrwxrwx 1 root    root    11137472 Jun  5 09:05 frps*
-rwxrwxrwx 1 root    root         156 Jun 14 14:55 frps.ini*
-rw-r--r-- 1 root    root        1742 Sep 20 17:00 log.txt
-rw-r--r-- 1 root    root       25680 Apr 27  2017 mysql57-community-release-el7-11.noarch.rpm
-rw-r--r-- 1 root    root       10923 Sep 17 18:07 openssl.cnf
drwxrwxrwx 2 root    root        4096 Jun 14 13:28 php-version/
-rwxrwxrwx 1 root    root          79 Aug 24 10:58 restart_frps.sh*
-rw-r--r-- 1 root    root         746 Sep 20 17:00 setting.json
drwxrwxrwx 2 shawvyu shawvyu     4096 Jul 26 15:05 shawvyu/

## 列出文件和文件夹（按大小排序 du -s * | sort -nr），大小是 KB。
[root@EMQ home]# du -s *
4	au.sh
68	certbot-auto
36	certs
3052	data_interconnection-x86_64.AppImage
18256	emqx-centos7-v3.0-beta.3.x86_64.rpm
19772	emqx-centos7-v3.2-beta.2.x86_64.rpm
10880	frps
4	frps.ini
4	log.txt
28	mysql57-community-release-el7-11.noarch.rpm
12	openssl.cnf
32	php-version
4	restart_frps.sh
4	setting.json
16	shawvyu
```

## 防火墙

### firewall

开启/关闭/查看防火墙/...

```shell
service start firewalld   # 开启
service stop firewalld    # 停止
service restart firewalld # 重启
service status firewalld  # 查看状态
service disable firewalld # 禁止开机启动
```

配置文件

```shell
# 系统配置目录
/usr/lib/firewalld/
/usr/lib/firewalld/services
/usr/lib/firewalld/zones

# 用户配置目录
/etc/firewalld/
/etc/firewalld/services
/etc/firewalld/zones
```

查看所有开放的端口、服务...

```shell
[root@cloud-q00zxm-bn25 ~]# firewall-cmd --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: ssh dhcpv6-client https
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

其余更多 firewall-cmd 命令可以参考：<https://wangchujiang.com/linux-command/c/firewall-cmd.html>

### iptables

参考：

- <https://wangchujiang.com/linux-command/c/iptables.html>
- <https://man.linuxde.net/iptables>

## IO CPU 内存 网络


### IO

**查看整体 IO 状况**

**查看单个进程 IO 使用状况**

### CPU

**查看整体 CPU 状况**
**查看单个进程 CPU 使用状况**

### 内存

**查看整体内存状况**

```shell
[root@vultr ~]# free -h
             total       used       free     shared    buffers     cached
Mem:          995M       364M       631M       472K        21M       270M
-/+ buffers/cache:        72M       923M
Swap:           0B         0B         0B
```

**查看单个进程内存使用状况**

### 网络

**查看整体网络状况**

nload 安装：`sudo yum install nload` 或者 `sudo apt install nload`。（参考：<https://linux.cn/article-2871-1.html>）

```shell
[root@vultr ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 56:00:02:75:20:7B  
          inet addr:66.42.61.217  Bcast:66.42.61.255  Mask:255.255.254.0
          inet6 addr: fe80::5400:2ff:fe75:207b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:71577 errors:0 dropped:0 overruns:0 frame:0
          TX packets:71248 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:67493280 (64.3 MiB)  TX bytes:9361011 (8.9 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:828 (828.0 b)  TX bytes:828 (828.0 b)

[root@vultr ~]# nload eth0

Device eth0 [66.42.61.217] (1/1):
===================================================================================================================================
Incoming:





                                                                                         Curr: 5.37 kBit/s
                                                                                         Avg: 5.16 kBit/s
                                                                                         Min: 328.00 Bit/s
                                                                                         Max: 9.91 kBit/s
                                                                                         Ttl: 64.37 MByte
Outgoing:






                                                                                         Curr: 14.03 kBit/s
                                                                                         Avg: 13.38 kBit/s
                                                                                         Min: 3.48 kBit/s
                                                                                         Max: 27.23 kBit/s
                                                                                         Ttl: 8.95 MByte
```

**查看单个进程网络使用状况**

nethogs 安装：`sudo yum install nethogs` 或者 `sudo apt install nethogs`。（参考：<https://linux.cn/article-2808-1.html>）

```shell
[root@vultr ~]# nethogs

NetHogs version 0.8.5

    PID USER     PROGRAM                                                                         DEV        SENT      RECEIVED       
   5885 root     sshd: root@pts/0,pts/1                                                          eth0        1.436       0.757 KB/sec
      ? root     unknown TCP                                                                                 0.000       0.000 KB/sec

  TOTAL                                                                                                      1.436       0.757 KB/sec
 
```
