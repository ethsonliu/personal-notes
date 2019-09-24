## 目录

- [查找](#查找)
  - [可执行文件](#可执行文件)
  - [进程](#进程)
  - [已安装程序](#已安装程序)
  - [普通文件](#普通文件)
- [查看某个进程的使用情况](#查看某个进程的使用情况)

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
ps -ef
```
```shell
## 按名字查找 ssh
ps -ef|grep ssh
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
ps -aux | grep frps
```
显示如下，
```
root      4340  0.0  1.1 113744 11984 pts/0    Sl   18:11   0:00 ./frps -c ./frps.ini
root      4375  0.0  0.0 112708   980 pts/0    R+   18:38   0:00 grep --color=auto frps
```
其中 【0.0=CPU %，【1.1】=内存 %，【11984】=内存KB

```shell
# 查看进程
cat /proc/2913/status
```
显示如下，
```
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

## 端口占用

查看服务器 8000 端口的占用情况：

```
# lsof -i:8000
COMMAND   PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
nodejs  26993 root   10u  IPv4 37999514      0t0  TCP *:8000 (LISTEN)
```

netstat -tunlp 用于显示 tcp，udp 的端口和进程等相关情况。

```
[root@iZ94xyihsxsZ home]# netstat -tunlp
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

例如查看 8000 端口的情况，使用以下命令：

```
# netstat -tunlp | grep 8000
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      26993/nodejs   
```

Linux查看程序端口占用情况
使用命令：

```
ps -aux | grep tomcat
```

## 查看磁盘和文件大小

使用`df -h`命令来查看磁盘信息， -h 选项为根据大小适当显示。

- **df -hl**：查看磁盘剩余空间
- **du -sh [目录名]**：返回该目录的大小
- **du -h --max-depth=1 [路径]**：查看该路径下所有第一层的文件夹的大小

按文件和文件夹大小排序`du -s * | sort -nr`，大小是 KB。

```
jack@jiaobuchong:~$ du -s * | sort -nr 
852756	installed-software
173868	Desktop
164768	Downloads
4724	Pictures
3236	program_pratice
452		Documents
284		learngit
112		session
12		examples.desktop
4		Videos
4		Templates
4		Public
```



