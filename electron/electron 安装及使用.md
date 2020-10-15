以下为 Windows 平台上的配置。

## 安装 Node.js

下载地址：<https://nodejs.org/en/>，最好下载稳定版本，然后安装，安装完成后，看下 PATH 路径是否已经把路径加入，如果没有，请手动加进去。

比如，我的安装目录是 `E:\Nodejs`，那么就把这个安装目录放进 PATH 中。

打开 cmd，验证是否安装成功，如下，

```shell
C:\Users\liuyi>node -v
v12.18.0
```

## 设置淘宝源

打开 cmd，输入 `npm config set registry https://registry.npm.taobao.org`，然后输入 `npm config get registry` 验证是否设置成功。


