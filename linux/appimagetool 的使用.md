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

