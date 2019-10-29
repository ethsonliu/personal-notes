## 目录

- [获取当前脚本所在绝对路径](#获取当前脚本所在绝对路径)
- [写入多行文件内容](#写入多行文件内容)

## 获取当前脚本所在绝对路径

```shell
#!/bin/bash

CURRENT_PATH=$(dirname $(readlink -f "$0"))
```

## 写入多行文件内容

```shell
#!/bin/bash

touch $DESKTOP_FILE

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
```

































