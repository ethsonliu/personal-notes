## 目录

- [读取文件所有内容](#读取文件所有内容)

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
