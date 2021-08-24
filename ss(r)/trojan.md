## 目录

- [Shadowsocks 是如何被检测和封锁的](#Shadowsocks-是如何被检测和封锁的)
- [Trojan 如何绕过 GFW](#Trojan-如何绕过-GFW)
- [服务器带宽测试](#服务器带宽测试)
  - [speedtest](#speedtest)
  - [iperf](#iperf)
- [acme 证书生成](#acme-证书生成)
- [参考](#参考)

## Shadowsocks-是如何被检测和封锁的

参考：<https://gfw.report/blog/gfw_shadowsocks/zh.html>

## Trojan 如何绕过 GFW

与 Shadowsocks 相反，Trojan 不使用自定义的加密协议来隐藏自身。相反，使用特征明显的 TLS 协议(TLS/SSL)，使得流量看起来与正常的 HTTPS 网站相同。TLS 是一个成熟的加密体系，HTTPS 即使用 TLS 承载 HTTP 流量。使用正确配置的加密 TLS 隧道，可以保证传输的

- 保密性（GFW 无法得知传输的内容）
- 完整性（一旦 GFW 试图篡改传输的密文，通讯双方都会发现）
- 不可抵赖（GFW 无法伪造身份冒充服务端或者客户端）
- 前向安全（即使密钥泄露，GFW 也无法解密先前的加密流量）

对于被动检测，Trojan 协议的流量与 HTTPS 流量的特征和行为完全一致。而 HTTPS 流量占据了目前互联网流量的一半以上，且 TLS 握手成功后流量均为密文，几乎不存在可行方法从其中分辨出 Trojan 协议流量。

对于主动检测，当防火墙主动连接 Trojan 服务器进行检测时，Trojan 可以正确识别非 Trojan 协议的流量。与 Shadowsocks 等代理不同的是，此时 Trojan 不会断开连接，而是将这个连接代理到一个正常的 Web 服务器。在 GFW 看来，该服务器的行为和一个普通的 HTTPS 网站行为完全相同，无法判断是否是一个 Trojan 代理节点。这也是 Trojan 推荐使用合法的域名、使用权威 CA 签名的 HTTPS 证书的原因: 这让你的服务器完全无法被 GFW 使用主动检测判定是一个 Trojan 服务器。

## 服务器带宽测试

### speedtest

```shell
cd /home
wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
chmod +x speedtest-cli
./speedtest-cli
```

输出如下，

```
Retrieving speedtest.net configuration...
Testing from DXTL Tseung Kwan O Service (154.204.50.208)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by STC (Hong Kong) [0.00 km]: 6.55 ms
Testing download speed................................................................................
Download: 4.61 Mbit/s
Testing upload speed................................................................................................
Upload: 2.11 Mbit/s
```

```shell
# 列出服务器列表
./speedtest-cli --list

# 选择指定服务器测试
./speedtest-cli --server 1204
```

### iperf

下载地址：https://iperf.fr/iperf-download.php

在服务端启动，

```shell
iperf3 -s -p 9999
```

在客户端启动，

```shell
# client-->server
.\iperf3.exe -c x.x.x.x -p 9999

# client<--server
.\iperf3.exe -c x.x.x.x -p 9999 -R
```

## acme 证书生成

```shell
# 如果执行超时，请参考 
# https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E
# https://github.com/acmesh-official/acme.sh
curl  https://get.acme.sh | sh

alias acme.sh=~/.acme.sh/acme.sh

export Ali_Key=""
export Ali_Secret=""

# 申请证书
acme.sh --issue --dns dns_ali -d example.com -d www.example.com

# 安装证书到指定目录
acme.sh --install-cert -d example.com \
--key-file       /usr/local/cert/ethsonliu.com/key.pem  \
--fullchain-file /usr/local/cert/ethsonliu.com/fullchain.pem \
--reloadcmd     "service nginx force-reload"

```

参考：<https://github.com/acmesh-official/acme.sh>

## 参考

- [自建梯子教程 --Trojan 版本](https://trojan-tutor.github.io/2019/04/10/p41.html)
- [trojan 教程](https://tlanyan.me/trojan-tutorial/)
