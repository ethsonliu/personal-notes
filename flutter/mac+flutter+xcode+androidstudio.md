mac 平台下进行 flutter 开发需要的配置全过程。

## flutter

到 https://docs.flutter.dev/get-started/install/macos 下载你要的版本。然后解压，将它的 bin 目录放进环境变量即可，我建议是放进 `/etc/paths` 文件中。

另外配置下国内的源，把

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

放在 `~/.bash_profile` 里。

## xcode

根据 https://developer.apple.com/cn/support/xcode/ 选择合适的 xcode 版本，下载并解压，移动到应用程序目录即安装完成。

## android studio

到 https://developer.android.com/studio 下载然后安装。

第一次可以选择配置 proxy，这个要配置一下，待会下载东西方便。

然后安装 flutter 插件，安装的过程也会让你安装 dart，一并装好就好。

## 测试有无全部配好

打开终端，输入 `flutter doctor`，就可以看到哪些没装好。

**Cocoa pods 的问题**

打开终端，输入 `sudo gem install cocoapods` 即可自动完成安装。

**flutter doctor 提示 command line tools not found**

打开 android studio —> preference —> android sdk —> sdk tools —> Android SDK Command-line Tools(latest) 点应用会自行下载安装。


## 参考

- https://blog.csdn.net/qq_27158179/article/details/120402279
