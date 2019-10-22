## 目录

- [读取文件内容](#读取文件内容)
  - [读取所有内容](#读取所有内容)
  - [一行一行读取](#一行一行读取)
- [写入文件内容](#写入文件内容)
  - [覆盖写入](#覆盖写入)
  - [尾部追加](#尾部追加)
  - [头部插入](#头部插入)

## 读取文件内容

### 读取所有内容

```c++
#include <fstream>
#include <sstream>
#include <string>

std::string readAllContent(const std::string &file)
{
    std::ifstream in;
    in.open(file);
    
    if (in.is_open())
    {
        std::stringstream stream;
        stream << in.rdbuf();
        return stream.str();
    }
    
    return std::string();
}
```

参考：<https://stackoverflow.com/questions/116038/what-is-the-best-way-to-read-an-entire-file-into-a-stdstring-in-c>

### 一行一行读取

```c++
#include <fstream>
#include <string>

void readLineByLine(const std::string &file)
{
    std::ifstream in;
    in.open(file);
    
    if (in.is_open())
    {
        std::string line;
        while (std::getline(in, line))
        {
            std::cout << line;
        }
    }
}
```

参考：<https://stackoverflow.com/questions/7868936/read-file-line-by-line-using-ifstream-in-c>

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
