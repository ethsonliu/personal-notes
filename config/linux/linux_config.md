目录

- [安装 VMware Tools](#安装-VMware-Tools)
- [改变下载源](#改变下载源)
- [安装 Gnome Tweak Tool](#安装-Gnome-Tweak-Tool)
- [安装 dash-to-panel 或者 dash-to-dock](#安装-dash-to-panel-或者-dash-to-dock)
- [桌面主题](#桌面主题)
- [图标主题](#图标主题)
- [字体](#字体)
- [安装 Qt](#安装-Qt)

## 安装 VMware Tools

```
tar -xzvf  VMwareTools-10.0.6-3595377.tar.gz
sudo ./wmware-install.pl
```

## 改变下载源

【设置】-【软件更新】，选择阿里云镜像站点

或者直接可以搜索到【软件更新器】


## 安装 Gnome Tweak Tool

```
sudo apt-get install gnome-tweak-tool gnome-shell-extensions
```

安装完成查看版本：`gnome-shell --version`，下面的安装尽量基于版本去下载，否则可能有问题。

## 安装 dash-to-panel 或者 dash-to-dock

dash-to-panel 像 windows 底部栏一样，dash-to-dock 像 mac 底部栏一样，

```
# https://extensions.gnome.org/extension/1160/dash-to-panel/
# https://extensions.gnome.org/extension/307/dash-to-dock/
# 解压，目录命名为：dash-to-panel@jderose9.github.com
# 解压，目录命名为：dash-to-dock@micxgx.gmail.com
# CTRL + H, 打开隐藏文件，放至 .local/share/gnome-shell/extensions/
# 重启电脑，打开 gnome-tweak-tool 查看扩展就可以发现 Dash to panel 或者 Dash to dock
```

## 桌面主题

github 地址：https://github.com/vinceliuice/Sierra-gtk-theme

下载地址：https://www.opendesktop.org/s/Gnome/p/1013714/

解压将里边的文件夹 mv 到 /usr/share/themes，重启电脑。

## 图标主题



## 字体

下载 Consolas 字体：https://github.com/EthsonLiu/personal-notes/blob/master/config/linux/YaHeiConsolas.tar.gz

```
tar -zxvf YaHeiConsolas.tar.gz
sudo mkdir -p /usr/share/fonts/vista
sudo cp YaHeiConsolas.ttf /usr/share/fonts/vista/
sudo chmod 644 /usr/share/fonts/vista/*.ttf
cd /usr/share/fonts/vista/
sudo mkfontscale && sudo mkfontdir && sudo fc-cache -fv
```

## 安装 Qt

原文链接：https://wiki.qt.io/Install_Qt_5_on_Ubuntu

```
chmod +x qt-opensource-linux-x64-5.7.0.run
./qt-opensource-linux-x64-5.7.0.run

# 安装 g++ 
sudo apt-get install build-essential

# 安装 generic font configuration library - runtime
sudo apt-get install libfontconfig1

# 安装 OpenGL (需要一条一条执行)
sudo apt-get install mesa-common-dev
sudo apt-get install libglu1-mesa-dev -y
```
