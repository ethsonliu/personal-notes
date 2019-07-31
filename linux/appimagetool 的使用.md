```
.
├── AppRun
├── data_interconnection
├── data_interconnection.png
├── data_interconnection.desktop
└── lib/
```

假设这个目录的名字叫`MyProject`，`appimage`一般我用来打包控制台应用程序，上面的文件逐一介绍，

- AppRun，是一个脚本，内容如下，

  ```
  #!/bin/bash
  HERE="$(dirname "$(readlink -f "${0}")")"
  export LD_LIBRARY_PATH=${HERE}/lib:$LD_LIBRARY_PATH
  exec "${HERE}/data_interconnection" "$@"
  ```

- data_interconnection，是我的可执行程序

- data_interconnection.png，是生成AppImage所需的图标

- data_interconnection.desktop，也是appimagetool所需要的，

  ```
  [Desktop Entry]
  Type=Application
  Name=data_interconnection
  Comment=data_interconnection
  Exec=data_interconnection
  Icon=data_interconnection
  Categories=Application;
  Terminal=true
  ```

- lib 目录，存放程序运行所需的库，其中把所需的库复制到lib目录，使用`ldd`命令，脚本为，（若有找不到所在库的路径的，该脚本执行自动会报错）

  ```
  #!/bin/sh
  
  exe="./data_interconnection"
  des="./lib"
  
  deplist=$(ldd $exe | awk  '{if (match($3,"/")){ printf("%s "),$3 } }')
  cp $deplist $des
  ```

好，至此一切就绪。

在该目录`MyProject`的同级目录下，打开终端，输入命令`appimagetool ./MyProject`，即可在同级目录看见`*.AppImage`文件，拷贝到其它机器上可以直接运行。

### 附可能遇到的问题：

#### 1. 在 ubuntu 14.04 上编译的，在 ubuntu 16.04 运行报错：段错误（核心已转存）

把 lib 目录里的的 glibc 库和 ANSI C 库去掉，重新生成即可。

```
libc.so
libgcc_s.so
libstdc++.so
libm.so
libpthread.so
还有其它的 glibc 库
```

下面是对一些库的介绍：

```
libm.so             // glibc 的数学库
libc.so             // Linux 下的 ANSI C 函数库
libgcc_s.so         // gcc 的底层运行时库，当一些操作在特定平台上不支持时，会自动生成对这些库函数的                       // 调用
libc++.so           // clang 编译器的 C++ 标准库
libstdc++.so        // gcc 编译器的 C++ 标准库
librt.so            // glibc 的 real-time 库
libpthread.so       // posix 线程库
```

#### 2. centos 7 上报错 dlopen(): error loading libfuse.so.2

报错，

```
dlopen(): error loading libfuse.so.2

AppImages require FUSE to run. 
You might still be able to extract the contents of this AppImage 
if you run it with the --appimage-extract option. 
See https://github.com/AppImage/AppImageKit/wiki/FUSE 
for more information
```

缺少 fuse 库，安装即可，`yum install fuse fuse-devel`。

