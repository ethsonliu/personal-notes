## 目录

- [consolas](#consolas)
- [终端](#终端)
- [访达显示当前目录路径](#访达显示当前目录路径)
- [vim 高亮](#vim-高亮)

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
