修改文件 /etc/ssh/sshd_config，

```
PermitRootLogin yes
PasswordAuthentication yes
```

可用命令 `sudo passwd root` 修改 root 密码。

最后重启服务 `service sshd restart`。
