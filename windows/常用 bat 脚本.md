## 目录

- [判断服务是否存在](#判断服务是否存在)
- [杀掉所有同名进程](#杀掉所有同名进程)
- [后台启动程序](#后台启动程序)

## 判断服务是否存在

```bat
REM Judge service mosquitto if is exited
SC QUERY mosquitto > NUL
IF ERRORLEVEL 1060 GOTO notexist
GOTO exist

REM If existed
:exist
echo service existed.
if "%one%" == "start" (
    echo service start.
    net start mosquitto
    %~dp0/mosquitto.exe -c %~dp0/../config/mosquitto.conf
) else (
    if "%one%"=="stop" (
        echo service stop.
        TASKKILL /F /IM mosquitto.exe
    ) else (
        echo Please input the correct parameter.
    )
)
REM Need exit here
exit

REM If not existed
:notexist
echo service not existed.
if "%one%" == "start" (
    echo service start.
    %~dp0/mosquitto.exe install
    net start mosquitto
    %~dp0/mosquitto.exe -c %~dp0/../config/mosquitto.conf
)
```

## 杀掉所有同名进程

```bat
TASKKILL /F /IM mosquitto.exe
```

## 后台启动程序

```bat
start /b "" "C:\Users\liuyi\Desktop\Scada-v3.21.0.9\Scada\Resources\app\bin\backmanage.exe"  "C:/Users/liuyi/Desktop/Scada-v3.21.0.9/Scada/Resources/app" "Online" >C:\Users\liuyi\Desktop\logout.txt 2>&1
```



