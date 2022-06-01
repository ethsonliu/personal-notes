提交的时候错误如下，

```
unable to access 'https://github.com/ethsonliu/typecho-theme-wab.git/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
```

在使用 HTTPS 连接 GitHub 进行 push/pull 时（即 origin 地址为 https://github.com/xxx/xxx.git ），需要更改本地 git 的配置，使用代理向 GitHub 发起请求。

执行如下命令：

```
git config --global -e
```

这将进入 git 的配置文件编辑界面（将使用 git 指定的默认编辑器打开）。

在该文件中加入如下内容：

```
[http]
        proxy = http://127.0.0.1:8080
[https]
        proxy = http://127.0.0.1:8080
```

因为我本地开的是 8080 的 http 代理，所以应该这写。

参考：

- https://blog.hyperzsb.tech/git-ssl-error/
