glibc 的英文全称为 The GNU C Library。

glibc 是 GNU发布的 libc 库，即 c 运行库。glibc 是 linux 系统中最底层的 api，几乎其它任何运行库都会依赖于 glibc。glibc 除了封装 linux 操作系统所提供的系统服务外，
它本身也提供了许多其它一些必要功能服务的实现。

## 直接安装

查看 libc.a 是否已经安装

```
sudo find / -name 'libc.a'
```

redhat/centos系列安装

```
sudo yum install glibc-static
```

debian/ubuntu系列安装

```
sudo apt-get install libc6-dev
```

## reference

- https://blog.csdn.net/itas109/article/details/104226783
