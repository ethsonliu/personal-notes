修改文件 /etc/ssh/sshd_config，

```
PermitRootLogin yes
PasswordAuthentication yes
```

最后重启服务 `service sshd restart`。
