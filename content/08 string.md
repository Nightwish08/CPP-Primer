* [std::string](https://en.cppreference.com/w/cpp/string/basic_string)

## 定义和初始化string对象
* 直接初始化和拷贝初始化
```cpp
string s("hello"); // 直接初始化
string s2 = "hello"; // 拷贝初始化
// 多个值的初始化尽量用直接初始化
string s3(10,'c');
string s4 = string(10,'c'); // 开销大
```
* char*可以直接转为string
```cpp
const char* s1 = "hello";
string s2(s1);
```
* string用c_str()转char*
```cpp
string s1("hello");
const char* s2 = s1.c_str();
```
## size_type和size_t
* size_type是string和vector的长度类型，标准库中定义为unsigned
* size_t是cstddef头文件中的类型，也是unsigned
* vector下标类型为vector::size_type，数组则是size_t
* std::string::npos代表size_t的最大值，因为size_t为unsigned，所以-1就是最大值
```cpp
static const size_t npos = -1;
```

## string对象上的操作
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
* cout一个string要包含string头文件
```cpp
string s = "hello";
cout << s; // 必须#include <string>才能用
```
* 读取未知数量的string对象
```cpp
int main()
{
    string word;
    while (cin >> word)
        cout << word << endl;
    return 0;
}
```
* getline用来读取一整行，`getline(is, s)`从给定的输入流读内容直到遇到换行符为止（只要一遇到换行符就结束读取并返回结果，即使一开始就是换行符，得到的结果是一个空string），换行符也被读进来了，只不过存到string对象里的时候不存换行符
```cpp
int main()
{
    string line;
    while (getline(cin, line))
        cout << line << endl; // line中不包含换行符，手动加上
    return 0;
}
```
* `string::size_type`类型是unsigned整型，如果一条表达式已经有了`size()`函数就不要用`int`，避免和带符号数混用，如假设n是一个`具有负值的int`，对`s.size() < n`来说会把n转换成一个比较大的无符号值，判断结果几乎只会是`true`
* 用加法运算符连接要保证两侧的运算对象至少有一个是string对象才会把另一个自动转换为string
```cpp
string s1 = "hello", s2 = "world";
string s3 = s1 + ", " + s2 + "\n";
string s4 = "hello" + ", " + s2; // 错误，"hello" + ", "
string s5 = s1 + ", " + "world"; // 正确
```

## 清空string
* clear删除内容使size为0，但capacity不变，还要用C++11的shrink_to_fit重置capacity使得capacity适应string大小
```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{
    string s("12345678901234567890");
    cout << s.size() << " " << s.capacity() << endl;
    s.clear(); // or s = ""
    cout << s.size() << " " << s.capacity() << endl;
    s.shrink_to_fit();
    cout << s.size() << " " << s.capacity() << endl;
    return 0;
}

// output
20 31
0 31
0 15
```

## 处理string对象中的字符
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
string str("some string");
for (auto c : str)
    cout << c << endl;
```
* 结合引用修改string对象中字符值
```cpp
string str("some string");
for (auto &c : str)
    c = toupper(c);
```
* 使用下标运算符访问置顶位置，下标必须大于等于0而小于s.size()，使用超出范围的下标会引发不可预知的结果（实际上下标为size()编译器不报错，因为string以'\0'结尾，其他超出范围会报错），而如果s为空，s[0]的结果是未定义的（其实是'\0'），所以在用下标前要先确认那个位置有值
```cpp
string s("some thing");
if(!s.empty())
    s[0] = toupper(s[0]);
```
* 把下标类型设为`string::size_type`确保下标不会小于0，此时代码只要保证下标小于size()
```cpp
// 把s的第一个词改成大写
for (decltype(s.size()) index = 0;
    index != s.size() && !isspace(s[index]); ++index)
    s[index] = toupper(s[index]);
```
* 子字符串
```cpp
// string substr (size_t pos = 0, size_t len = npos) const;
string s("hello world");
string s2 = s.substr(0, 5); // 从位置0开始的5个字符的拷贝，hello
string s3 = s.substr(6); // world
string s4 = s.substr(6, 100); // world
string s5 = s.substr(11); // s5为空
string s6 = s.substr(12); // 抛出out_of_range异常
size_t pos = str.find("world"); // pos = 6
string s7 = str.substr(pos); // world
```
