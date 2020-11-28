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
- [netstat](#netstat)
  - [端口占用](#端口占用)
  - [查看 tcp 状态](#查看-tcp-状态)
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

## 查看某个进程启动的个数

```shell
[root@iZ94xyihsxsZ home]# ps -ef|grep php-fpm|grep -v "grep"|wc -l
6
```

## netstat

参考：

- [https://www.runoob.com/linux/linux-comm-netstat.html](https://www.runoob.com/linux/linux-comm-netstat.html)
- [https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html](https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html)

### 端口占用

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

### 查看 tcp 状态

```shell
hapoa@hapoa-virtual-machine:~$ netstat -nat
激活Internet连接 (服务器和已建立连接的)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:8084            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.1.1:53            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:53049           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:5369            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:60506           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:1883            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:46205           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:6369            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:18083           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:11883         0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:53709           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:4369            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8883            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8083            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:45235           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:38203         127.0.0.1:4369          ESTABLISHED
tcp        0      0 127.0.0.1:4369          127.0.0.1:38203         ESTABLISHED
tcp6       0      0 :::59230                :::*                    LISTEN     
tcp6       0      0 :::2049                 :::*                    LISTEN     
tcp6       0      0 :::52195                :::*                    LISTEN     
tcp6       0      0 :::51331                :::*                    LISTEN     
tcp6       0      0 127.0.0.1:4332          :::*                    LISTEN     
tcp6       0      0 :::47855                :::*                    LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::59440                :::*                    LISTEN     
tcp6       0      0 :::4369                 :::*                    LISTEN
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

systemctl disable firewalld # 禁止开机启动
systemctl enable firewalld  # 允许开机启动
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

```shell
# 永久加入端口
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp

# 不中断服务的重新加载
firewall-cmd --reload

# 中断所有连接的重新加载
firewall-cmd --complete-reload      

# 删掉端口
firewall-cmd --remove-port=9999/tcp
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

先整体观察：

```shell
[root@iZ94xyihsxsZ ~]# vmstat 1 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 700448 200044 1517300    0    0     6    25    0    1  5  4 90  0  0
 0  0      0 699984 200044 1517300    0    0     0    32 6066 8901  9  9 83  0  0
 0  0      0 700060 200044 1517308    0    0     0     0 6259 9544  9  7 84  0  0
 1  0      0 700232 200044 1517308    0    0     0   180 6054 9223 10  5 83  1  0
 0  0      0 699812 200044 1517276    0    0     0     0 5669 8258  5  7 88  0  0

```

- Procs（进程）
  - r：运行队列中进程数量，这个值也可以判断是否需要增加 CPU，长期大于 1
  - b：等待 IO 的进程数量
- Memory（内存）
  - swpd：使用虚拟内存大小，如果 swpd 的值不为 0，但是 SI，SO 的值长期为 0，这种情况不会影响系统性能
  - free：空闲物理内存大小
  - buff：用作缓冲的内存大小
  - cache：用作缓存的内存大小，如果 cache 的值大的时候，说明 cache 处的文件数多，如果频繁访问到的文件都能被 cache 处，那么磁盘的读 IO bi 会非常小
- Swap
  - si：每秒从交换区写到内存的大小，由磁盘调入内存
  - so：每秒写入交换区的内存大小，由内存调入磁盘

注意：内存够用的时候，这 2 个值都是 0，如果这 2 个值长期大于 0 时，系统性能会受到影响，磁盘 IO 和 CPU 资源都会被消耗。有些朋友看到空闲内存（free）很少的或接近于 0 时，就认为内存不够用了，不能光看这一点，还要结合 si 和 so，如果 free 很少，但是 si 和 so 也很少（大多时候是 0），那么不用担心，系统性能这时不会受到影响的。因为 linux 总是先把内存用光。

- IO
  - bi：每秒读取的块数
  - bo：每秒写入的块数
- system（系统）
  - in：每秒中断数，包括时钟中断
  - cs：每秒上下文切换数
- CPU（以百分比表示）
  - us：用户进程执行时间百分比(user time) us 的值比较高时，说明用户进程消耗的 CPU 时间多，但是如果长期超 50% 的使用，那么我们就该考虑优化程序算法或者进行加速
  - sy：内核系统进程执行时间百分比(system time) sy 的值高时，说明系统内核消耗的 CPU 资源多，这并不是良性表现，我们应该检查原因
  - wa：IO 等待时间百分比 wa 的值高时，说明 IO 等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈（块操作）
  - id：空闲时间百分比

### IO

**查看整体 IO 状况**

```shell
[root@EMQ ~]# iostat -xm 2
Linux 3.10.0-957.5.1.el7.x86_64 (EMQ)   12/17/2019      _x86_64_        (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.24    0.00    2.32    0.09    0.00   94.35

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     1.41    0.01    1.37     0.00     0.01    19.92     0.00    1.89    6.08    1.86   0.79   0.11

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.06    0.00    4.55    0.00    0.00   89.39

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     2.02    0.00    1.52     0.00     0.02    26.67     0.00    1.00    0.00    1.00   0.67   0.10

```

%util 代表磁盘繁忙程度。

**查看单个进程 IO 使用状况**

安装：`sudo apt install iotop` 或者 `sudo yum install iotop`。

```shell
[root@EMQ ~]# iotop
Total DISK READ :       0.00 B/s | Total DISK WRITE :       0.00 B/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                                                                                             
    1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % systemd --switched-root --system --deserialize 22
    2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]
    3 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/0]
    5 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/0:0H]
    7 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/0]
    8 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_bh]
    9 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_sched]
   10 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [lru-add-drain]
   11 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/0]
   13 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kdevtmpfs]
   14 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [netns]
   15 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [khungtaskd]
   16 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [writeback]
   17 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kintegrityd]
   18 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [bioset]
   19 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [bioset]
   20 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [bioset]
   21 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kblockd]
   22 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [md]
   23 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [edac-poller]
   24 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdogd]
 3098 be/4 mysql       0.00 B/s    0.00 B/s  0.00 %  0.00 % mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
 3247 be/4 mysql       0.00 B/s    0.00 B/s  0.00 %  0.00 % mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
 3162 be/2 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % AliYunDun
   30 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kswapd0]
   31 be/5 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksmd]
   32 be/7 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [khugepaged]
   33 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [crypto]
   41 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthrotld]
   43 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kmpath_rdacd]
   44 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kaluad]
   45 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kpsmoused]
   46 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ipv6_addrconf]
11784 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % frps -c /home/frps.ini
 3251 be/2 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % AliYunDun
  777 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ttm_swap]

```

### CPU

**查看整体 CPU 状况**

us + sy = 整体 cpu

```shell
[root@iZ94xyihsxsZ ~]# vmstat 1 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 700448 200044 1517300    0    0     6    25    0    1  5  4 90  0  0
 0  0      0 699984 200044 1517300    0    0     0    32 6066 8901  9  9 83  0  0
 0  0      0 700060 200044 1517308    0    0     0     0 6259 9544  9  7 84  0  0
 1  0      0 700232 200044 1517308    0    0     0   180 6054 9223 10  5 83  1  0
 0  0      0 699812 200044 1517276    0    0     0     0 5669 8258  5  7 88  0  0

```

**查看单个进程 CPU 使用状况（前 10）**

```shell
[root@iZ94xyihsxsZ ~]# ps aux|head -1;ps aux|grep -v PID|sort -rn -k +3|head -10
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root     22510 10.1  2.5 227216 99232 ?        Sl   Dec11 237:00 /root/ngrok/bin/ngrokd -domain=machine.haiwell.com -httpAddr=:8000 -log-level=ERROR -log=/root/ngrok/log/running.log -httpsAddr=:4433
mysql    25823  5.6 15.9 1465556 617356 ?      Sl   Nov01 3401:55 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock
root     12753  2.3  0.2 131184  9204 ?        S<sl Nov19 817:18 /usr/local/aegis/aegis_client/aegis_10_75/AliYunDun
root      4100  1.8  5.1 1382988 200908 ?      Rsl  Dec12  19:53 node /var/local/cloud2/dist/index.js
root        36  1.3  0.0      0     0 ?        S    Aug24 2171:10 [kswapd0]
root      8152  0.8  0.5 114008 19912 ?        Sl   Dec12   8:33 ./frps -c frps.ini
root     16350  0.5  0.1 155596  6296 ?        Ss   10:30   0:00 sshd: root@pts/0,pts/1
root       547  0.4  0.5 1016988 22816 ?       Ssl  Aug24 705:10 /usr/local/cloudmonitor/CmsGoAgent.linux-amd64
nginx     5326  0.2  0.2 126660  8960 ?        S    Dec02  37:13 nginx: worker process
nginx     5325  0.2  0.2 125680  8252 ?        S    Dec02  44:27 nginx: worker process

```

或者更简单点，直接`top`再按`shift + P`，

```shell
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
25823 mysql     20   0 1465556 617356   4368 S  19.3 15.9   3402:10 mysqld                                                                                                                                         
22510 root      20   0  227216  99040   3856 S  12.0  2.6 237:14.24 ngrokd                                                                                                                                         
 4100 root      20   0 1392716 214416  13636 S   4.0  5.5  19:56.34 node /var/local                                                                                                                                
12753 root      10 -10  131184   9204   5008 S   3.3  0.2 817:21.83 AliYunDun                                                                                                                                      
 8152 root      20   0  114008  19912   6944 S   1.0  0.5   8:35.17 frps                                                                                                                                           
 5325 nginx     20   0  125680   8252   2400 S   0.7  0.2  44:28.21 nginx                                                                                                                                          
16350 root      20   0  155596   6308   4588 S   0.7  0.2   0:01.10 sshd                                                                                                                                           
    9 root      20   0       0      0      0 S   0.3  0.0 291:42.38 rcu_sched                                                                                                                                      
  547 root      20   0 1016988  22760   5364 S   0.3  0.6 705:11.09 CmsGoAgent.linu                                                                                                                                
 1076 redis     20   0  142960   5072    716 S   0.3  0.1 182:25.21 redis-server                                                                                                                                   
 1398 root      20   0  989532  90572   3952 S   0.3  2.3 169:41.56 PM2 v2.4.6: God                                                                                                                                
 5326 nginx     20   0  126660   8960   2484 S   0.3  0.2  37:13.90 nginx                                                                                                                                          
21618 root      20   0 1234324  46300  12064 S   0.3  1.2   1:05.90 node /var/local                                                                                                                                
    1 root      20   0  190996   2612   1316 S   0.0  0.1  15:41.54 systemd                                                                                                                                        
    2 root      20   0       0      0      0 S   0.0  0.0   0:01.82 kthreadd                                                                                                                                       
    3 root      20   0       0      0      0 S   0.0  0.0  72:08.33 ksoftirqd/0                                                                                                                                    
    5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                                                                                                                                   
    7 root      rt   0       0      0      0 S   0.0  0.0   2:10.56 migration/0                                                                                                                                    
    8 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcu_bh                                                                                                                                         
   10 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 lru-add-drain                                                                                                                                  
   11 root      rt   0       0      0      0 S   0.0  0.0   0:45.77 watchdog/0                                                                                                                                     
   12 root      rt   0       0      0      0 S   0.0  0.0   0:35.94 watchdog/1                                                                                                                                     
   13 root      rt   0       0      0      0 S   0.0  0.0   2:01.02 migration/1                                                                                                                                    
   14 root      20   0       0      0      0 S   0.0  0.0   6:09.66 ksoftirqd/1                                                                                                                                    
   16 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/1:0H                                                                                                                                   
   18 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kdevtmpfs                                                                                                                                      
   19 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 netns                                                                                                                                          
   20 root      20   0       0      0      0 S   0.0  0.0   0:03.49 khungtaskd                                                                                                                                     
   21 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 writeback                                                                                                                                      
   22 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kintegrityd                                                                                                                                    
   23 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 bioset                                                                                                                                         
   24 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 bioset 
```

### 内存

**查看整体内存状况**

```shell
[root@vultr ~]# free -h
             total       used       free     shared    buffers     cached
Mem:          995M       364M       631M       472K        21M       270M
-/+ buffers/cache:        72M       923M
Swap:           0B         0B         0B
# 995M = 72M + 923M

[root@iZ94xyihsxsZ ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.4G        684M         48M        1.6G        2.0G
Swap:            0B          0B          0B
# 3.7G = 1.6G + 2.0G
```

含义参考：<https://blog.51cto.com/ixdba/541355>

**查看单个进程内存使用状况**

列出内存消耗排名靠前的进程（以下是列出前 10）：

```shell
[root@vultr ~]# ps aux|head -1;ps aux|grep -v PID|sort -rn -k +4|head -10
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root     23258  0.0  0.4  98252  4644 ?        Ss   00:57   0:05 sshd: root@pts/0,pts/1
root     28138  0.0  0.3  96676  3836 ?        Ss   02:26   0:00 sshd: unknown [priv]
root      1200  0.0  0.3  81036  3464 ?        Ss   Dec12   0:00 /usr/libexec/postfix/master
root      1017  0.0  0.3 251392  3220 ?        Sl   Dec12   0:05 /sbin/rsyslogd -i /var/run/syslogd.pid -c 5
postfix  27363  0.0  0.3  81116  3424 ?        S    01:17   0:00 pickup -l -t fifo -u
postfix   1207  0.0  0.3  81284  3472 ?        S    Dec12   0:00 qmgr -l -t fifo -u
root     23273  0.0  0.2  57300  2168 ?        Ss   00:57   0:00 /usr/libexec/openssh/sftp-server
ntp       1120  0.0  0.2  30740  2112 ?        Ss   Dec12   0:00 ntpd -u ntp:ntp -p /var/run/ntpd.pid -g
sshd     28139  0.0  0.1  67632  1772 ?        S    02:26   0:00 sshd: unknown [net]
root     28142  0.0  0.1 110240  1172 pts/0    R+   02:26   0:00 ps aux

```

如果想确定一个进程内部具体的内存使用情况，可以用下面的命令：

```shell
[root@vultr ~]# cat /proc/1214/status
Name:   crond
State:  S (sleeping)
Tgid:   1214
Pid:    1214
PPid:   1
TracerPid:      0
Uid:    0       0       0       0
Gid:    0       0       0       0
Utrace: 0
FDSize: 64
Groups: 
VmPeak:   116896 kB
VmSize:   116876 kB
VmLck:         0 kB
VmHWM:      1292 kB
VmRSS:      1288 kB
VmData:      880 kB
VmStk:       520 kB
VmExe:        60 kB
VmLib:      2064 kB
VmPTE:        64 kB
VmSwap:        0 kB
Threads:        1
SigQ:   1/3898
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: 0000000000000000
SigCgt: 0000000000014003
CapInh: 0000000000000000
CapPrm: ffffffffffffffff
CapEff: ffffffffffffffff
CapBnd: ffffffffffffffff
Speculation_Store_Bypass:       vulnerable
Cpus_allowed:   1
Cpus_allowed_list:      0
Mems_allowed:   00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:      0
voluntary_ctxt_switches:        1391
nonvoluntary_ctxt_switches:     3
[root@vultr ~]# pmap -x 1214
1214:   crond
Address           Kbytes     RSS   Dirty Mode   Mapping
0000562f5206d000      60      44       0 r-x--  crond
0000562f5227b000       4       4       4 r----  crond
0000562f5227c000       4       4       4 rw---  crond
0000562f5227d000     512     136     136 rw---    [ anon ]
0000562f52413000     132      32      32 rw---    [ anon ]
00007f290cd8a000      52      20       0 r-x--  libnss_files-2.12.so
00007f290cd97000    2044       0       0 -----  libnss_files-2.12.so
00007f290cf96000       4       4       4 r----  libnss_files-2.12.so
00007f290cf97000       4       4       4 rw---  libnss_files-2.12.so
00007f290cf98000   96848       4       0 r----  locale-archive
00007f2912e2c000       8       4       0 r-x--  libfreebl3.so
00007f2912e2e000    2044       0       0 -----  libfreebl3.so
00007f291302d000       4       4       4 r----  libfreebl3.so
00007f291302e000       4       4       4 rw---  libfreebl3.so
00007f291302f000      28       4       0 r-x--  libcrypt-2.12.so
00007f2913036000    2048       0       0 -----  libcrypt-2.12.so
00007f2913236000       4       4       4 r----  libcrypt-2.12.so
00007f2913237000       4       4       4 rw---  libcrypt-2.12.so
00007f2913238000     184       0       0 rw---    [ anon ]
00007f2913266000    1580     448       0 r-x--  libc-2.12.so
00007f29133f1000    2044       0       0 -----  libc-2.12.so
00007f29135f0000      16      16      16 r----  libc-2.12.so
00007f29135f4000       8       8       8 rw---  libc-2.12.so
00007f29135f6000      16      16      16 rw---    [ anon ]
00007f29135fa000      96       8       0 r-x--  libaudit.so.1.0.0
00007f2913612000    2044       0       0 -----  libaudit.so.1.0.0
00007f2913811000       8       8       8 r----  libaudit.so.1.0.0
00007f2913813000      44       8       8 rw---  libaudit.so.1.0.0
00007f291381e000       8       4       0 r-x--  libdl-2.12.so
00007f2913820000    2048       0       0 -----  libdl-2.12.so
00007f2913a20000       4       4       4 r----  libdl-2.12.so
00007f2913a21000       4       4       4 rw---  libdl-2.12.so
00007f2913a22000      48       8       0 r-x--  libpam.so.0.82.2
00007f2913a2e000    2048       0       0 -----  libpam.so.0.82.2
00007f2913c2e000       4       4       4 r----  libpam.so.0.82.2
00007f2913c2f000       4       4       4 rw---  libpam.so.0.82.2
00007f2913c30000     116      40       0 r-x--  libselinux.so.1
00007f2913c4d000    2044       0       0 -----  libselinux.so.1
00007f2913e4c000       4       4       4 r----  libselinux.so.1
00007f2913e4d000       4       4       4 rw---  libselinux.so.1
00007f2913e4e000       4       4       4 rw---    [ anon ]
00007f2913e4f000     128      84       0 r-x--  ld-2.12.so
00007f2914065000      20      20      20 rw---    [ anon ]
00007f291406e000       4       4       4 rw---    [ anon ]
00007f291406f000       4       4       4 r----  ld-2.12.so
00007f2914070000       4       4       4 rw---  ld-2.12.so
00007f2914071000       4       4       4 rw---    [ anon ]
00007ffe32d70000     520     296     296 rw---    [ stack ]
00007ffe32df4000       4       4       0 r-x--    [ anon ]
ffffffffff600000       4       0       0 r-x--    [ anon ]
----------------  ------  ------  ------
total kB          116880    1288     616

```

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
