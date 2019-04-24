如果 Ubuntu 是装在虚拟机，注意使用桥接模式。

安装服务器端，

```
sudo apt-get install nfs-kernel-server
```

修改配置文件，

```
sudo gedit /etc/exports
```

末尾加入，`/home/work/nfs *(rw,sync,no_root_squash,no_subtree_check)`，目录不存在就自己创建，注意这个权限改为 777。

其中各参数意思为，

/home/work/nfs，与nfs服务客户端共享的目录
*，允许所有的网段访问，也可以使用具体的IP
rw，挂接此目录的客户端对该共享目录具有读写权限
sync，资料同步写入内存和硬盘
no_root_squash，root用户具有对根目录的完全管理访问权限
no_subtree_check，不检查父目录的权限

接着重启，

```
# 重启 portmap，后面最新的版本都rpcbind重启
sudo /etc/init.d/rpcbind restart

# 重启 nfs 服务
sudo /etc/init.d/nfs-kernel-server restart

# 显示共享出的目录
showmount -e
```

另一台嵌入式设备执行，

```
mount 192.168.201.134:/home/work/nfs /work/nfs -o nolock
```

即可。

## 参考

- https://blog.csdn.net/u010346967/article/details/46384641
- https://www.linuxidc.com/Linux/2016-04/129848.htm