### 安装 VMware Tools

```
tar -xzvf  VMwareTools-10.0.6-3595377.tar.gz
sudo ./wmware-install.pl
```

### 删掉不必要应用（仅个人而言）

```
sudo apt-get remove thunderbird empathy simple-scan gnome-mahjongg aisleriot gnome-mines gnome-sudoku onboard
```

### 改变源

【设置】-【软件更新】，选择阿里云镜像站点

### 安装 gnome-tweak-tool

```
sudo apt-get install gnome-tweak-tool
sudo apt-get install gnome-shell-extensions

# 安装 dash-to-panel
# 1: https://extensions.gnome.org/extension/1160/dash-to-panel/
# 2: 解压至 home
# 3: CTRL + H, 打开隐藏文件，放至 .local/share/gnome-shell/extensions/
# 4: 重启
```

### 图标主题

开源地址：[https://github.com/PapirusDevelopmentTeam/papirus-icon-theme](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme)

下载下来解压，将 `Papirus` 几个目录，复制到`/usr/share/icons/`下。

再到第 2 步安装的 gnome-tweak-tool 中启用。

### 桌面主题

 - [arc-theme](https://github.com/NicoHood/arc-theme)
 - [materia-theme](https://github.com/nana-4/materia-theme)
 - [adapta-gtk-theme](https://github.com/adapta-project/adapta-gtk-theme)

### 字体

下载 Consolas 字体：https://github.com/Hapoa/_Repository/blob/master/YaHeiConsolas.tar.gz

```
tar -zxvf YaHeiConsolas.tar.gz
sudo mkdir -p /usr/share/fonts/vista
sudo cp YaHeiConsolas.ttf /usr/share/fonts/vista/
sudo chmod 644 /usr/share/fonts/vista/*.ttf
cd /usr/share/fonts/vista/
sudo mkfontscale && sudo mkfontdir && sudo fc-cache -fv
```

### 安装 Qt

原文链接：https://wiki.qt.io/Install_Qt_5_on_Ubuntu

```
chmod +x qt-opensource-linux-x64-5.7.0.run
./qt-opensource-linux-x64-5.7.0.run

# 安装 g++ 
sudo apt-get install build-essential

# 安装 generic font configuration library - runtime
sudo apt-get install libfontconfig1

# 安装 OpenGL
sudo apt-get install mesa-common-dev
sudo apt-get install libglu1-mesa-dev -y
```
