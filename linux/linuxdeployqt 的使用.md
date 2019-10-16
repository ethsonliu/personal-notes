假设当前准备打包的目录为`MyProject`，

```
.
├── Data Interconnection
├── Data Interconnection.png
├── Data Interconnection.desktop
```

`Data Interconnection`是我的准备打包发布的可执行程序。

在里边新建`Data Interconnection.png`和`Data Interconnection.desktop`，

其中`Data Interconnection.desktop`内容如下，

```
[Desktop Entry]
Type=Application
Name=Data Interconnection
Comment=Data Interconnection
Exec=Data Interconnection
Icon=Data Interconnection
Categories=Application;
Terminal=true
```

然后进入MyProject目录，执行`linuxdeployqt ./'Data Interconnection' -appimage -always-overwrite -no-copy-copyright-files -qmake='/opt/5.10.1/gcc_64/bin/qmake'`，目录结构变为，

```
./
├── AppRun -> Data Interconnection
├── Data Interconnection
├── Data_Interconnection-068c258-x86_64.AppImage
├── default.desktop
├── default.png
├── lib/
├── plugins/
│   ├── imageformats/
│   ├── platforminputcontexts/
│   ├── platforms/
│   └── xcbglintegrations/
├── qt.conf
└── translations/
```

`Data_Interconnection-068c258-x86_64.AppImage`就是你想要的。（如果你在MyProject的父级目录运行打包命令，则AppImage就生成在父级目录中）

**2019-10-16 补充：**

用 desktop 启动器来更好，linuxdeployqt 用来拉取所需配置。

add_menuitem.sh

```shell
#!/bin/bash

CURRENT_PATH=$(dirname $(readlink -f "$0"))

ICON_NAME=iotbus-data-interconnection
DESKTOP_FILE=iotbus-data-interconnection.desktop
QT_CONF_FILE=qt.conf

touch $DESKTOP_FILE
touch $QT_CONF_FILE

cat << EOF > $DESKTOP_FILE
[Desktop Entry]
Encoding=UTF-8
Name=Data Interconnection
Keywords=MQTT
Comment=A data interconnection tool
Type=Application
Categories=Development
Terminal=false
Exec="$CURRENT_PATH/start.sh"
Icon=$ICON_NAME.png
EOF

cat << EOF > $QT_CONF_FILE
[Paths]
Plugins = $CURRENT_PATH/../plugins
EOF

chmod 777 $DESKTOP_FILE
chmod 777 $ICON_NAME.png
chmod 777 $QT_CONF_FILE

# xdg-desktop-menu install $DESKTOP_FILE 
# 这条语句同时会复制内容到 ~/.local/share/applications/，所以改用 cp
cp $DESKTOP_FILE /usr/share/applications

xdg-icon-resource install --size 128 "$ICON_NAME.png" $ICON_NAME

rm $DESKTOP_FILE

```

remove_menuitem.sh

```shell
#!/bin/bash

xdg-desktop-menu uninstall iotbus-data-interconnection.desktop
xdg-icon-resource uninstall --size 128 iotbus-data-interconnection
```

start.sh

```shell
#!/bin/bash

CURRENT_PATH=$(dirname $(readlink -f "$0"))

LIB_PATH=$CURRENT_PATH/../lib

export LD_LIBRARY_PATH=$LIB_PATH:$LD_LIBRARY_PATH

exec "$CURRENT_PATH/Data Interconnection"
```


