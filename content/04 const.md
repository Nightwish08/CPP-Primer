* const对象**一旦创建就不能改变**，所以const对象**必须初始化**
```cpp
const int i; // 错误
const int* p; // 正确，p不是const，只是指向const所以可以不初始化
const int* const p; // 错误
```
* 默认情况下，const对象只在文件内有效，在每个文件中对const变量不管是声明还是定义都添加extern，则只需要定义一次
```cpp
// file_1.cc，该常量能被其他文件访问
extern const int bufSize = f();
// file_1.h，和file_1.cc定义的变量是同一个
extern const int bufSize;
```
* 用指针指向常量（pointer to const），不能用于改变其所指对象的值
```cpp
const double p = 3.14;
double* ptr = &p; // 错误，ptr是普通指针
const double* cptr = &p;
*cptr = 42; // 错误，不能给*cptr赋值
```
* const pointer必须初始化，初始化后指针（存在其中的地址）不能再改变，即不能指向其他对象
```cpp
int i = 0;
int* const p = &i; // p将一直指向i
const double d = 3.14;
const double* const p2 =  &d; // p2是指向const的const pointer
*p = 1; // 此时i也为1
i = 2; // 此时*p也为2
*p2 = 2.72; // 错误
```
* 从右向左读，p最近的符号是const，表示p是const对象，再是*，说明p是一个const pointer，它指向int，同理，p2是const pointer，指向的是const double对象
* **不能把const引用或指针赋给non-const**，反过来可以
```cpp
int i = 12;
const int &r = i;
const int *p = &i;
int &r2 = r; // 错误
int *q = p; // 错误
```
* 不能改的称为顶层const（top-level const），可改的称为底层 const（low-level const）
```cpp
int i = 0;
int* const p1 = &i; // 不能改变p1的值，顶层const
const int ci = 42; // 不能改变ci的值，顶层const
const int* p2 = &c1; // 允许改变p2的值，底层const
const int* const p3 = p2; // 第一个const是底层const，第二个是顶层const
const int& r = ci; // 用于声明引用的const都是底层const
```
* const对一条语句多个对象都有作用
```cpp
const int i = 0, j = 1;
j = 2; // 错误
const int ic, &r = ic; // 错误，ic未初始化
```
* const expresssion指值不会改变并且在编译时就能得到计算结果的表达式
```cpp
const int sz = get_sezi(); // 不是const表达式，因为具体值要到运行时才能取到
```
* C++11中，允许将变量声明为constexpr类型让编译器来验证变量的值是否是一个常量表达式，声明为constexpr的变量一定是一个常量，而且必须用常量表达式初始化。如果你确定变量是一个常量表达式，那就把它声明为constexpr类型，这样做的好处是
  * 更好地保证程序的正确语义
  * 编译器可以对constexpr进行很大的优化，如把用到的const expression直接替换为结果
  * 比起宏没有额外开销
```cpp
constexpr int mf = 20; // 20是常量表达式
constexpr int limit = mf + 1; // mf+1是常量表达式
constexpr int sz = size(); // 只有当size是一个constexpr函数时才是正确的声明语句
```
* constexpr会把对象置为顶层const，指针比较特殊，constexpr仅对指针本身有效，与指针所指的对象无关
```cpp
const int* p = nullptr; // p是一个指向const int的指针
constexpr int* q = nullptr;// q是一个指向int的const指针

constexpr int i = 42;
int j = 0;
constexpr const int *p = &i; // p是指向const int i的const pointer
constexpr const int *q = &j; // q是指向int j的const pointer
```
