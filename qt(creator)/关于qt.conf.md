这个文件指定了 qt 程序的运行环境，即我们可以使用 qt.conf 指定 qt 程序的运行路径和库路径。内容如下所示：

```
# Generated by linuxdeployqt
# https://github.com/probonopd/linuxdeployqt/
[Paths]
Prefix = ./		#程序的运行路径
Libraries =  ./lib  #程序的库路径
Plugins = plugins	#插件路径
Imports = qml
Qml2Imports = qml
```

见：https://doc.qt.io/qt-5/qt-conf.html
