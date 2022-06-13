## 目录

- [homebrew](#homebrew)
- [consolas](#consolas)
- [终端](#终端)
- [访达显示当前目录路径](#访达显示当前目录路径)
- [vim 高亮](#vim-高亮)

## homebrew

```shell
cd ~
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install >> brew_install
vim brew_install
```

修改为

```
#!/System/Library/Frameworks/Ruby.framework/Versions/Current/usr/bin/ruby
# This script installs to /usr/local only. To install elsewhere you can just
# untar https://github.com/Homebrew/brew/tarball/master anywhere you like or
# change the value of HOMEBREW_PREFIX.
HOMEBREW_PREFIX = "/usr/local".freeze
HOMEBREW_REPOSITORY = "/usr/local/Homebrew".freeze
HOMEBREW_CACHE = "#{ENV["HOME"]}/Library/Caches/Homebrew".freeze
HOMEBREW_OLD_CACHE = "/Library/Caches/Homebrew".freeze
#BREW_REPO = "https://github.com/Homebrew/brew".freeze
BREW_REPO = "https://mirrors.aliyun.com/homebrew/brew.git".freeze # 这里
#CORE_TAP_REPO = "https://github.com/Homebrew/homebrew-core".freeze
CORE_TAP_REPO = "https://mirrors.aliyun.com/homebrew/homebrew-core.git".freeze # 这里可能已经没有了
```

修改好之后，按 ESC，输入 :wq 保存退出，执行 brew_install，

```shell
/usr/bin/ruby ~/brew_install
```

1. 替换 homebrew 默认源

```shell
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
```

2. 替换 homebrew-core 源

```shell
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
```

3. 更新 brew

```shell
brew update
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
