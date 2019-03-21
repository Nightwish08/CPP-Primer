* [std::string](https://en.cppreference.com/w/cpp/std::string/basic_std::string)

## 定义和初始化std::string对象
* 直接初始化和拷贝初始化
```cpp
std::string s("hello"); // 直接初始化
std::string s2 = "hello"; // 拷贝初始化
// 多个值的初始化尽量用直接初始化
std::string s3(10,'c');
std::string s4 = std::string(10,'c'); // 开销大
```
* char*可以直接转为std::string
```cpp
const char* s1 = "hello";
std::string s2(s1);
```
* std::string用c_str()转char\*
```cpp
std::string s1("hello");
const char* s2 = s1.c_str();
```
## size_type和std::size_t
* size_type是std::string和std::vector的长度类型，标准库中定义为unsigned
* std::size_t是cstddef头文件中的类型，也是unsigned
* std::vector下标类型为std::vector::size_type，数组则是std::size_t
* std::std::string::npos代表std::size_t的最大值，因为std::size_t为unsigned，所以-1就是最大值
```cpp
static const std::size_t npos = -1;
```

## std::string对象上的操作
```cpp
os << s;
is >> s;
getline(is, s);
s.front(); // C++11
s.back(); // C++11
s.empty();
s.size();
s.capacity();
s.clear();
s.c_str();
s.shrink_to_fit(); // C++11
s[n];
s1 + s2;
s1 = s2;
s1 == s2;
s1 != s2;
<, <=, >, >=
```
* std::cout一个std::string要包含std::string头文件
```cpp
std::string s = "hello";
std::cout << s; // 必须#include <string>才能用
```
* 读取未知数量的std::string对象
```cpp
int main()
{
    std::string word;
    while (cin >> word) std::cout << word << '\n';
    return 0;
}
```
* getline用来读取一整行，`getline(is, s)`从给定的输入流读内容直到遇到换行符为止（只要一遇到换行符就结束读取并返回结果，即使一开始就是换行符，得到的结果是一个空std::string），换行符也被读进来了，只不过存到std::string对象里的时候不存换行符
```cpp
int main()
{
    std::string line;
    while (getline(cin, line))
        std::cout << line << '\n'; // line中不包含换行符，手动加上
    return 0;
}
```
* `std::string::size_type`类型是unsigned整型，如果一条表达式已经有了`size()`函数就不要用`int`，避免和带符号数混用，如假设n是一个`具有负值的int`，对`s.size() < n`来说会把n转换成一个比较大的无符号值，判断结果几乎只会是`true`
* 用加法运算符连接要保证两侧的运算对象至少有一个是std::string对象才会把另一个自动转换为std::string
```cpp
std::string s1 = "hello", s2 = "world";
std::string s3 = s1 + ", " + s2 + "\n";
std::string s4 = "hello" + ", " + s2; // 错误，"hello" + ", "
std::string s5 = s1 + ", " + "world"; // 正确
```

## 清空std::string
* clear删除内容使size为0，但capacity不变，还要用C++11的shrink_to_fit重置capacity使得capacity适应std::string大小
```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{
    std::string s("12345678901234567890");
    std::cout << s.size() << " " << s.capacity() << '\n';
    s.clear(); // or s = ""
    std::cout << s.size() << " " << s.capacity() << '\n';
    s.shrink_to_fit();
    std::cout << s.size() << " " << s.capacity() << '\n';
    return 0;
}

// output
20 31
0 31
0 15
```

## 处理std::string对象中的字符
* cctype头文件包含了一组检测字符特性的函数
```cpp
isalnum(c); // 数字或字母
isalpha(c); // 字母
iscntrl(c); // 控制字符
isdigit(c); // 数字
isgraph(c); // 不是空格但可打印
islower(c); // 小写字母
isprint(c); // 是可打印字符（即c是空格或具有可视形式）
ispunct(c); // 标点
isspace(c); // 空白（空格、制表符、回车符、换行符等）
isupper(c); // 大写字母
isxdigit(c); // 十六进制数字
tolower(c); // 转小写
toupper(c); // 转大写
```
* 用范围for语句遍历
```cpp
std::string str("some std::string");
for (auto c : str)
    std::cout << c << '\n';
```
* 结合引用修改std::string对象中字符值
```cpp
std::string str("some std::string");
for (auto &c : str)
    c = toupper(c);
```
* 使用下标运算符访问置顶位置，下标必须大于等于0而小于s.size()，使用超出范围的下标会引发不可预知的结果（实际上下标为size()编译器不报错，因为std::string以'\0'结尾，其他超出范围会报错），而如果s为空，s[0]的结果是未定义的（其实是'\0'），所以在用下标前要先确认那个位置有值
```cpp
std::string s("some thing");
if(!s.empty())
    s[0] = toupper(s[0]);
```
* 把下标类型设为`std::string::size_type`确保下标不会小于0，此时代码只要保证下标小于size()
```cpp
// 把s的第一个词改成大写
for (decltype(s.size()) index = 0;
    index != s.size() && !isspace(s[index]); ++index)
    s[index] = toupper(s[index]);
```
* 子字符串
```cpp
// std::string substr (std::size_t pos = 0, std::size_t len = npos) const;
std::string s("hello world");
std::string s2 = s.substr(0, 5); // 从位置0开始的5个字符的拷贝，hello
std::string s3 = s.substr(6); // world
std::string s4 = s.substr(6, 100); // world
std::string s5 = s.substr(11); // s5为空
std::string s6 = s.substr(12); // 抛出out_of_range异常
std::size_t pos = str.find("world"); // pos = 6
std::string s7 = str.substr(pos); // world
```
