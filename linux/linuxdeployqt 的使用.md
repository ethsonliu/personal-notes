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

`ata_Interconnection-068c258-x86_64.AppImage`就是你想要的。（如果你在MyProject的父级目录运行打包命令，则AppImage就生成在父级目录中）