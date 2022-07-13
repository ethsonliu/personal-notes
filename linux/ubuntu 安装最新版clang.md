## 获取签名

```
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
```

## 添加 llvm 软件源地址

pls refer to https://apt.llvm.org/

choose your ubuntu version, if it's ubuntu18, pls add the following text to `/etc/apt/sources.list`.

```
deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic main
deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic main
# Needs 'sudo add-apt-repository ppa:ubuntu-toolchain-r/test' for libstdc++ with C++20 support
# 13
deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-13 main
deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic-13 main
# 14
deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-14 main
deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic-14 main
```

## 安装

```
sudo apt update
sudo apt install clang
clang -v
```
