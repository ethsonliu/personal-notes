## 环境预置

以下放在 `/etc/profile` 里，

```bash
export GOROOT=/home/hapoa/packages/local/go-1-14-2
export GOPROXY=https://goproxy.cn
export PATH=$PATH:/home/hapoa/packages/local/go-1-14-2/bin
```

下面这个可以不放，每次编译的时候再指定
export GOPATH=/home/hapoa/packages/caddy-master
export GO111MODULE=on

## 工程创建

Go 1.11 中的 module 支持临时环境变量 GO111MODULE，它可以设置以下三个值：off，on 或者 auto(默认)。

- 如果 GO111MODULE=off，那么 go 命令行将不会使用新的 module 功能，相反的，它将会在 vendor 目录下和 GOPATH 目录中查找依赖包。也把这种模式叫 GOPATH 模式。
- 如果 GO111MODULE=on，那么 go 命令行就会使用 modules 功能，而不会访问 GOPATH。也把这种模式称作 module-aware 模式，这种模式下，GOPATH 不再在 build 时扮演导入的角色，但是尽管如此，它还是承担着存储下载依赖包的角色。它会将依赖包放在 GOPATH/pkg/mod 目录下。
- 如果 GO111MODULE=auto，这种模式是默认的模式，也就是说在你不设置的情况下，就是 auto。这种情况下，go 命令行会根据当前目录来决定是否启用 module 功能，只有当当前目录在 GOPATH/src 目录之外而且当前目录包含 go.mod 文件或者其子目录包含 go.mod 文件才会启用。

以上参考：<https://blog.csdn.net/benben_2015/article/details/82227338>

个人习惯 module-aware 模式。

假设现有一个工程，工程名为 my_project，

```bash
# 切换到 cmd 下 main.go 的同级目录，然后执行以下

GOPATH=/home/hapoa/packages/caddy-master

CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build -o ./release/frpc_darwin_amd64 ./cmd/frpc

CGO_ENABLED=0 GOOS=freebsd GOARCH=386 go build -o ./release/frpc_freebsd_386 ./cmd/frpc

CGO_ENABLED=0 GOOS=freebsd GOARCH=amd64 go build -o ./release/frpc_freebsd_amd64 ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=386 go build -o ./release/frpc_linux_386 ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ./release/frpc_linux_amd64 ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=arm go build -o ./release/frpc_linux_arm ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o ./release/frpc_linux_arm64 ./cmd/frpc

CGO_ENABLED=0 GOOS=windows GOARCH=386 go build -o ./release/frpc_windows_386.exe ./cmd/frpc

CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o ./release/frpc_windows_amd64.exe ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=mips64 go build -o ./release/frpc_linux_mips64 ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=mips64le go build -o ./release/frpc_linux_mips64le ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=mips GOMIPS=softfloat go build -o ./release/frpc_linux_mips ./cmd/frpc

CGO_ENABLED=0 GOOS=linux GOARCH=mipsle GOMIPS=softfloat go build -o ./release/frpc_linux_mipsle ./cmd/frpc
```

