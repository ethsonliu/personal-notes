当前使用的 qt 版本：qt-opensource-windows-x86-msvc2015-5.6.3.exe，但注意这个 <https://blog.csdn.net/libaineu2004/article/details/80415755>

`.pro` 文件加入 `QMAKE_LFLAGS_WINDOWS = /SUBSYSTEM:WINDOWS,5.01`，参考 <https://blog.csdn.net/qq527703883/article/details/54925632>

如果有用到 openssl，那么编译的时候下面的代码，否则会报错 bcrypt.dll 找不到。


