## 目录

- [homebrew](#homebrew)
- [consolas](#consolas)
- [终端](#终端)
- [访达显示当前目录路径](#访达显示当前目录路径)
- [vim 高亮](#vim-高亮)

## homebrew

1. 安装 homebrew，https://github.com/Homebrew/brew/releases ，下载 pkg 安装包，安装完 `brew --version` 测试是否安装，不成功需要加入环境变量，见下。
2. (可选) `whereis brew` 查看所在位置，并加入 `/etc/proile`。

安装成功，设置国内源，

```
# 替换各个源
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git

# 打开 /etc/profile 并加入 export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
source /etc/profile

# 刷新源
$ brew update
```
**可能遇到的问题**

```
ethson@macbookpro:~$ brew update
Error:
  homebrew-core is a shallow clone.
  homebrew-cask is a shallow clone.
To `brew update`, first run:
  git -C /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core fetch --unshallow
  git -C /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask fetch --unshallow
These commands may take a few minutes to run due to the large size of the repositories.
This restriction has been made on GitHub's request because updating shallow
clones is an extremely expensive operation due to the tree layout and traffic of
Homebrew/homebrew-core and Homebrew/homebrew-cask. We don't do this for you
automatically to avoid repeatedly performing an expensive unshallow operation in
CI systems (which should instead be fixed to not use shallow clones). Sorry for
the inconvenience!
```

解决办法：

```
git -C "/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core" fetch
git -C "/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core" fetch --unshallow
git -C "/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask" fetch
git -C "/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask" fetch --unshallow
```

## consolas

参考：<https://gist.github.com/avalonalex/8125197>

```shell
# Thanks to this post:
# http://blog.ikato.com/post/15675823000/how-to-install-consolas-font-on-mac-os-x

brew install cabextract
cd ~/Downloads
mkdir consolas
cd consolas
curl -O https://sourceforge.net/projects/mscorefonts2/files/cabs/PowerPointViewer.exe
cabextract PowerPointViewer.exe
cabextract ppviewer.cab
open CONSOLA*.TTF
```

## 终端


```shell
chsh -s /bin/bash # 更改用户登录 shell
sudo chmod o+w /etc/profile # 赋予写权限
vi /etc/profile
```

末尾加入以下内容：

```shell
export LS_OPTIONS='--color=auto'
export CLICOLOR='Yes'
export LSCOLORS='CxfxcxdxbxegedabagGxGx'

if [[ $UID == 0 ]]; then
   export PS1='\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;31m\]\u@\h\[\033[00m\]:\[\033[01;35m\]\w\[\033[00m\]\$ '
else
   export PS1='\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
fi
```

保存退出 vi，执行 `source /etc/profile`，电脑重启也生效。

$UID == 0 说明是 root。

当进入 root 用户时，你会发现颜色都变了（红色），为了区分普通用户（绿色）和 root 用户，需要在颜色上改变一下，以醒目提醒操作者当前是以 root 执行命令。

注意在切换到 root 的时候要以登录的方式，不能 `sudo su`，要 `sudo -i`，因为 `sudo su` 的颜色依旧不会生效。

## 访达显示当前目录路径

```shell
defaults write com.apple.finder _FXShowPosixPathInTitle -bool YES
```

## vim 高亮

```shell
cp /usr/share/vim/vimrc ~/.vimrc
vi ~/.vimrc
```

加入以下几句，保存退出即可。

```
syntax on
set autoindent
```
