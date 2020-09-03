## 目录

- [字体](#字体)
- [QPushButton](#QPushButton)
- [QDockWidget](#QDockWidget)
- [QTableView](#QTableView)
- [](#)
- [](#)
- [](#)
- [其它链接](#其它链接)

## 字体

```c++
// 设置全局字体
QFont font;
font.setFamily("MS Shell Dlg 2");
qApp->setFont(font);

// 输出当前系统下的所有字体
QFontDatabase database;
foreach (const QString &family, database.families())
{
    qDebug() << family;
}

// 检测全局字体：
qDebug() << qApp->font().rawName();
qDebug() << qApp->font().family();
qDebug() << qApp->font().defaultFamily();
qDebug() << qApp->font().styleName();
qDebug() << qApp->font().toString();
qDebug() << qApp->font().key();
```

## QPushButton

```css

```

## QDockWidget

参考：

- <https://github.com/danielepantaleone/eddy>
- <https://blog.csdn.net/wzs250969969/article/details/78466143>

## QTableView

参考：

- <https://github.com/lowbees/Hover-entire-row-of-QTableView>

## 其它链接

- [Bootstrap](https://www.runoob.com/bootstrap4/bootstrap4-tutorial.html)
- [Qt Style Sheets Examples](https://doc.qt.io/qt-5/stylesheet-examples.html)
- [https://github.com/satchelwu/QSS-Skin-Builder](https://github.com/satchelwu/QSS-Skin-Builder)
- [https://github.com/chenwen1126/Qss](https://github.com/chenwen1126/Qss)
