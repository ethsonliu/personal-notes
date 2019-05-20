目录名，均是小写英文字母，多个单词，以下划线连接，不允许以`s`结尾来表示`多个`的含义，单词一律以单数形式呈现。例如，

```
third_party
bin
lib
dll
runtime
example
include
doc
locale
image
test
......
```

## 其它：

#### 1. 尽量不使用 resource 文件夹

尽量以`image, stylesheet, locale...`等具体资源名称代替`resource`这样的统一名称。


#### 2. third_party 目录安排

对于`third_party`下的库的目录安排，应当将`PWD/third_party`目录放进工程`path`中，包含库的头文件时，需指定库的名称，以防止库文件名冲突，例如`#include <cJOSN/cJSON.h>`。另外，里边放的库的名称该是大写就大写。各个库包含的内容只是需要`include`的头文件，不包括实现文件、编译好的库文件等等。