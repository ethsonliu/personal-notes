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

切换至 zsh，并

```
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

然后修改主题(到 https://github.com/ohmyzsh/ohmyzsh/wiki/Themes 找喜欢的主题),

```
vim ~/.zshrc

ZSH_THEME="gallois"
```

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
