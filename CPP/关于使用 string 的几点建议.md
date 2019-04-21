## 1. 使用 += 运算符以避免产生临时对象

```c++
std::string name("Jack");

std::string name = "I am" + name; // 应该用下面的

std::string word;
word += "I am";
word += name;
```

## 2. reserve

若知道大概占用多少内存，则需要`reserve`，避免多次分配内存，

```c++
std::string exec;
exec.reserve(1024);
```

## 3. 使用迭代器遍历元素

```c++
std::string s;

for (int i = 0; i < s.length(); ++i) // 效率比下面的会低
{
	s[i];
}

for (auto it=s.begin(),end=s.end(); it != end; ++it) // 建议用这个
{
    *it;
}
```

