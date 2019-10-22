## 目录

- [读取文件所有内容](#读取文件所有内容)
- [写入文件内容](#写入文件内容)
  - [覆盖写入](#覆盖写入)
  - [尾部追加](#尾部追加)
  - [头部插入](#头部插入)

## 读取文件所有内容

```c++
#include <fstream>
#include <sstream>
#include <string>

std::string readAllContent(const std::string &file)
{
    std::ifstream in;
    in.open(file);
    
    std::stringstream stream;
    stream << in.rdbuf();
    
    return stream.str();
}
```

## 写入文件内容

### 覆盖写入

```c++

```

### 尾部追加

```c++

```

### 头部插入

```c++

```
