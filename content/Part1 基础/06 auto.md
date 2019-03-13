* auto让编译器通过初始值分析表达式所属的类型，因此**auto定义时必须有初始值**
* auto一般会忽略顶层const保留底层const，简单理解就是直接用auto定义的非指针或引用类型都是non-const
```cpp
int i = 0;
const int ci = i, &cr = ci;
auto b = ci; // b是int，ci的顶层const特性被忽略
auto c = cr; // c是int，cr只不过是ci的别名
auto d = &i; // d是int*
auto e = &ci; // e是const int*（对常量对象取地址是一种底层const）
```
* 如果希望auto推断出顶层const，加上const修饰符
```cpp
const int ci = i;
const auto f = ci; // ci的推演类型是int，f是const int
```
* 可以将引用的类型设置为auto，相当于用auto替代原来的类型定义，遵循原来的初始化规则，const会保留
```cpp
int i = 0;
const int &ci = i;
auto& j = ci; // const int &j
auto& k = 42; // 错误，引用必须初始化
const auto& r = 42; // const int &r
// 设置auto类型的引用，初始值的顶层const属性保留
```
* 对于universal reference，只有右值才会将auto&&推断为右值引用
```cpp
int i = 42;
const int j = 55;
auto&& r1 = i; // int& r1
auto&& r2 = j; // const int& r2
auto&& r3 = 45; // int&& r3
```
* 对数组名，auto会推断为指针，而auto&会推断为数组
```cpp
int a[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
auto a2(a); // a2是int*，和auto a2(&a[0])等价
cout << a2[2]; // 正确，用指针同样可以获取元素
for(auto x : a2) cout << x; // 错误
auto& a3(a); // int(&)[10]
for(auto x : a3) cout << x; // 正确
```
* 也可以用decltype来推断为数组
```cpp
decltype(a) a4 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
```
* 在声明符列表中，auto必须始终推导为同一类型
```cpp
auto i = 1, j = 1.1; // 错误
```
