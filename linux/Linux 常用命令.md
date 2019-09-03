## 目录

- [查找](#查找)
  - [which](#which)
  - [whereis](#whereis)
  
## 查找

### which

查看可执行文件的位置。例如，

```shell
[root@~]# which openssl
/usr/bin/openssl
[root@~]# which make
/usr/bin/make
```

### whereis


## 按名字查找进程

```
# 按名字查找 ssh
ps -ef|grep ssh

# 显示所有进程，带执行它的命令行
ps -ef
```

参考：https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/ps.html

## 查看某个进程的使用情况

```
# 按 pid 查看
top -p 2913

# ps 指令
ps -aux | grep frps
# 显示如下，
# root      4340  0.0  1.1 113744 11984 pts/0    Sl   18:11   0:00 ./frps -c ./frps.ini
# root      4375  0.0  0.0 112708   980 pts/0    R+   18:38   0:00 grep --color=auto frps
# 其中 【0.0=CPU %，【1.1】=内存 %，【11984】=内存KB

# cat 指令，按 pid
cat /proc/2913/status
# 其中，VmRSS=内存KB
```

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



