## SOURCES 和 HEADERS

指定项目中的所有源文件和头文件。

最简单的用法是这样的，

```
SOURCES += main.cpp\
           mainwindow.cpp \
    	   serialtab.cpp \
           connectorsdlg.cpp \
           serialdataset.cpp \
           serialotherset.cpp \

HEADERS += mainwindow.h \
    	   serialtab.h \
    	   connectorsdlg.h \
    	   serialdataset.h \
    	   serialotherset.h \
```

如果你的工程有多个文件夹，每个文件内又有多个子文件夹，下面又有子文件夹，那上面的写法就有点鸡肋了，可以换成这样，

```
SOURCES += $$files(*.cpp, true)
HEADERS += $$files(*.h, true)
```

`files`是一个函数，按第一个参数去匹配，返回文件列表。第二个参数为 true时，表示子文件夹递归搜索。