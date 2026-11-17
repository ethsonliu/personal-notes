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

vscode settings json add below,

```
"workbench.colorCustomizations": {
        "terminal.background": "#292A44",
        "terminal.foreground": "#F1EFF8",
        "terminalCursor.background": "#F1EFF8",
        "terminalCursor.foreground": "#F1EFF8",
        "terminal.ansiBlack": "#292A44",
        "terminal.ansiBlue": "#2DE0A7",
        "terminal.ansiBrightBlack": "#666699",
        "terminal.ansiBrightBlue": "#2DE0A7",
        "terminal.ansiBrightCyan": "#8EAEE0",
        "terminal.ansiBrightGreen": "#6DFEDF",
        "terminal.ansiBrightMagenta": "#7AA5FF",
        "terminal.ansiBrightRed": "#A0A0C5",
        "terminal.ansiBrightWhite": "#53495D",
        "terminal.ansiBrightYellow": "#AE81FF",
        "terminal.ansiCyan": "#8EAEE0",
        "terminal.ansiGreen": "#6DFEDF",
        "terminal.ansiMagenta": "#7AA5FF",
        "terminal.ansiRed": "#A0A0C5",
        "terminal.ansiWhite": "#F1EFF8",
        "terminal.ansiYellow": "#AE81FF"
    }
```


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
