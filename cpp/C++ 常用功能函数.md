## 目录

- [读取文件所有内容](#读取文件所有内容)

## 读取文件所有内容

```c++
#include <fstream>
#include <sstream>
#include <string>

string readAllContent(string file)
{
    ifstream in;
    in.open(file);
    
    stringstream stream;
    stream << in.rdbuf();
    
    return stream.str();
}
```
