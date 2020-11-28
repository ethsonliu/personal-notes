/etc/sysctl.conf 中增加一行

```shell
net.ipv4.icmp_echo_ignore_all=1
```

如果已经有 net.ipv4.icmp_echo_ignore_all 这一行了，直接修改 = 号后面的值即可的（0 表示允许，1 表示禁止）。

修改完成后执行 `sysctl -p` 使新配置生效。
