```
# 1. Windows x86 桌面应用程序
#DEFINES += \
#    DESKTOP_APPLICATION \
#    OS_WINDOWS

# 2. Linux x64 桌面应用程序
DEFINES += \
    DESKTOP_APPLICATION \
    OS_LINUX

# 3. Linux x64 控制台应用程序
#DEFINES += \
#    CONSOLE_APPLICATION \
#    OS_LINUX

TEMPLATE = app

INCLUDEPATH += \
    $$PWD/third_party

LIBS += \
    -L$$PWD/third_party/mosquitto/lib -lmosquitto \
    -L$$PWD/third_party/cJSON/lib -lcJSON

SOURCES += \
    src/main.cpp \
    src/storage.cpp \
    src/user_data.cpp \
    src/storage_manager.cpp

HEADERS += \
    src/information_repeater.h \
    src/config.h \
    src/storage_manager.h \
    src/storage.h \
    src/user_data.h

contains(DEFINES, DESKTOP_APPLICATION) {
QT += core gui
TARGET = MQTTClient
greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

RC_ICONS = image/haiwell_mqtt.ico

TRANSLATIONS += \
    translation/zh_CN.ts \
    translation/en_US.ts

SOURCES += \
    src/main_window.cpp \
    src/table_widget.cpp \
    src/mqtt_server_dialog.cpp \
    src/setting_dialog.cpp \
    src/information_repeater.cpp

HEADERS += \
    src/main_window.h \
    src/table_widget.h \
    src/mqtt_server_dialog.h \
    src/setting_dialog.h

RESOURCES += \
    translation/translation.qrc
} # contains(DEFINES, DESKTOP_APPLICATION)

contains(DEFINES, CONSOLE_APPLICATION) {
CONFIG += console
CONFIG -= app_bundle
CONFIG -= qt
}

contains(DEFINES, OS_WINDOWS) {
LIBS += \
    -L$$PWD/third_party/MySQL/lib -llibmysql
}

contains(DEFINES, OS_LINUX) {
LIBS += \
    -L$$PWD/third_party/MySQL/lib -lmysqlclient \
    -lpthread
}
```

