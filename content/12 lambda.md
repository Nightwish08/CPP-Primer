* lambda表达式表示一个可调用的代码单元，可以理解为未命名的内联函数，一个lambda具有一个返回类型、一个参数列表和一个函数体，与普通函数不同的是，lambda可能定义在函数内部，且必须使用尾置返回类型
```cpp
[capture list] (parameter list) -> return type { function body }
// capture list 是lambda所在函数中定义局部变量的列表，通常为空
```
* 可以忽略参数列表和返回类型，但必须包含捕获列表和函数体
```cpp
auto f = [] { return 42; };
```
* 调用方式和普通函数相同，都是使用调用运算符
```cpp
std::cout << f() << std::endl;
```
* 忽略括号和参数列表等价于空参数列表，忽略返回类型则根据函数体推断，如果只是一个return语句则从返回的表达式类型推断，如果包含任何单一return语句之外的内容且未指定返回类型则返回void。可能出现这样的问题
```cpp
transform(vi.begin(), vi.end(), [] (int i) { return i < 0 ? -i : i; });
// 改写成等价的if语句则会出错
transform(vi.begin(), vi.end(),
    [] (int i) { if (i < 0) return -i; else return i; });
// 上式看似没问题，但编译器认为返回void类型，实际返回int
```
* **lambda不能有默认参数**，所以一个lambda调用的实参数等于形参数，一旦形参初始化完毕就可以执行函数体
```cpp
[] (const string &a, const string &b)
    { return a.size() < b.size(); }
// 调用
stable_sort(words.begin(), words.end(),
    [] (const string &a, const string &b) { return a.size() < b.size(); });
```
* lambda通过将局部变量包含在其捕获列表中来指明将会使用这些变量，**只有在捕获列表中捕获所在函数中的局部变量才能在函数体内使用，捕获列表只用于局部非static变量，lambda可以直接使用局部static变量和它所在函数之外声明的名字**
```cpp
// 编写一个可以传递给find_if的可调用表达式
// 我们希望它将输入序列中每个string的长度与biggies函数中的sz参数的值进行比较
[sz] (const string &a)
    { return a.size() >= sz; }; // 必须捕获sz函数体才能使用sz
// 使用此lambda，可以查找第一个长度大于等于sz的元素
auto wc = find_if(words.begin(), words.end(),
    [sz] (const string &a)
        { return a.size() >= sz; });
// 对find_if的调用返回指向第一个长度不小于sz的元素的迭代器  
// 如果不存在此元素则返回words.end()的一个拷贝
// 可以用此迭代器计算它到words末尾一共有多少个元素
auto count = words.end() - wc;
cout << " " << make_plural(count, "word", "s")
    << "of length " << sz << " or longer" << endl;
// make_plural根据count大小是否为1输出"word"或者"words"
// 最后用for_each算法打印长度大于等于sz的单词，每个单词后接一个空格
for_each (wc, words.end(),
    [] (const string &s) { cout << s << " "; });
```
* 完整的biggies
```cpp
void elimDups(vector<string> &words)
{
    sort(words.begin(), words.end());
    auto end_unique = unique(words.begin(), words.end());
    // unique只是把相邻重复元素移到了末尾，返回的是重复元素的首地址
    words.erase(end_unique, words.end());
}
void biggies(vector<string> &words, vector<string>::size_type sz)
{
    elimDups(words); // 按字典序排序，删除重复单词
    stable_sort(words.begin(), words.end(),
        [] (const string &a, const string &b)
            { return a.size() < b.size(); });
    auto wc = find_if(words.begin(), words.end(),
        [sz] (const string &a)
            { return a.size() >= sz; });
    auto count = words.end() - wc;
    cout << " " << make_plural(count, "word", "s")
        << "of length " << sz << " or longer" << endl;
    for_each (wc, words.end(),
        [] (const string &s) { cout << s << " "; });
    cout << endl;
}
```
* 用lamda捕获所在函数的int并接受一个int，返回和
```cpp
int sum(int a, int b)
{
    auto f = [a] (int b) { return a + b; };
    return f(b);
}
```
* lambda值捕获在创建时拷贝而不是调用时
```cpp
void fcn1()
{
    size_t v1 = 42;
    auto f = [v1] { return v1; };
    v1 = 0;
    auto j = f(); // j为42
}
```
* 捕获指针、迭代器、引用必须保证在lambda执行时对象存在，且要保证对象具有预期的值。在捕获时对象的值是我们所期望的，但在lambda从创建到执行的这段时间内可能有代码改变绑定对象的值
```cpp
void fcn2()
{
    size_t v1 = 42;
    auto f = [&v1} { return v1; };
    v1 = 0;
    auto j = f2(); // j为0
}
```
* ostream对象不能拷贝，可以用引用捕获
```cpp
void biggies(vector<string> &words, vector<string>::size_type sz,
    ostream &os = cout, char c = ' ')
{
    for_each(words.begin(), words.end(),
        [&os, c] (const string &s) { os << s << c; });
}
```
* 尽量减少捕获的数据量避免潜在风险，尽量避免捕获指针或引用  
* **在捕获列表中写一个&或=对所有变量采用隐式捕获，&代表引用捕获，=代表值捕获。两者可以混用以对部分值采用某种捕获方式，C++11规定混用时捕获列表中的第一个元素必须指定为&或=以确定默认隐式捕获方式**
```cpp
wc = find_if(words.begin(), words.end(),
    [=] (const string &s)
        { return s.size() >= sz; }); // 隐式值捕获sz
void biggies(vector<string> &words, vector<string>::size_type sz,
    ostream &os = cout, char c = ' ')
{
    for_each(words.begin(), words.end(),
        [=, &os] (const string &s) { os << s << c; }); // 显式引用捕获os，隐式值捕获c
}
```
* lambda默认不会改变值被拷贝的变量，如果要改变捕获变量的值要用mutable将其声明为可变lambda，可变lambda可以省略参数列表。但如果值捕获的是const变量，可变lambda也不能对其进行修改
```cpp
void fcn3() {
    size_t v1 = 42;
    auto f = [v1] () mutable { return ++v1; };
    v1 = 0;
    auto j = f(); // j为43
}
```
* 引用捕获的变量只要不是const就可以被修改
```cpp
void fcn4() {
    size_t v1 = 42;
    auto f2 = [&v1] { return ++v1; };
    v1 = 0;
    auto j = f2(); // j为1
}
```
