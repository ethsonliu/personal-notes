#### 按名字查找进程

```
# 按名字查找 ssh
ps -ef|grep ssh

# 显示所有进程，带执行它的命令行
ps -ef
```

参考：https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/ps.html

#### 查看某个进程的使用情况

```
# 按 pid 查看
top -p 2913

# ps 指令
ps -aux | grep kafka
# 显示如下，
# root      4340  0.0  1.1 113744 11984 pts/0    Sl   18:11   0:00 ./frps -c ./frps.ini
# root      4375  0.0  0.0 112708   980 pts/0    R+   18:38   0:00 grep --color=auto frps
# 其中 【0.0=CPU %，【1.1】=内存 %，【11984】=内存KB

# cat 指令，按 pid
cat /proc/2913/status
# 其中，VmRSS=内存KB
```

