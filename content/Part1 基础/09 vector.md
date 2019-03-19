* 定义vector对象的常用方法
```cpp
vector<T> v1;
vector<T> v2(v1);
vector<T> v2 = v1;
vector<T> v3(n, val); // v3包含n个val
vector<T> v4(n); // v4包含n个执行了值初始化的对象
vector<T> v5 {a, b, c...}
vector<T> v5 = {a, b, c...}
```
* 其他vector操作
```cpp
v.empty();
v.size();
v.push_back(t);
v[n];
v1 = v2;
v1 = {a, b, c...}
v1 == v2;
v1 != v2;
```
* 迭代器类型分为iterator、const_iterator，`begin()和end()`返回类型由对象是否是const决定，要专门得到const_iterator则可以使用cbegin和cend
* `end`指示的是容器的一个本不存在的尾后元素，`end`返回的迭代器称为尾后迭代器或尾迭代器
```cpp
vector<int> i (10,1);
for(auto it = v.begin(); it != v.end(); ++it)
```
* **v.begin() ！= v.end()说明非空，v.begin() == v.end()说明空**
* 用数组初始化vector，只需要传入两个指针确定范围
```cpp
int arr[] = {0, 1, 2, 3, 4, 5};
vector<int> v(begin(arr), end(arr));
vector<int> subVec(arr + 1, arr + 4); // arr[1]、arr[2]、arr[3]
```
* 删除一个vector里的指定的元素
```cpp
vector<int> v{2, 3, 5, 2, 6};
vector<int>::iterator it = v.begin();
while (it != v.end())
{
    if (*it == 2) it = v.erase(it);
    else ++it;
}
```
* 利用列表初始化返回vector
```cpp
vector<string> f()
{
    // s是一个string对象
    return {"functionX", "okay", s};
}
```
* vector元素类型不能是引用，因为指向引用的指针（迭代器）是非法的
```cpp
vector<int&> v; // error
```
* 可以保存指针，但指针必须先初始化再添加到容器
```cpp
int* p;
vector<int*> v { p }; // 错误：使用了未初始化的局部变量
v[0] = new int(42); // 错误
```
* vector添加元素实际执行的是拷贝操作，即保存的指针与原始对象已经无关了
```cpp
int* p;
vector<int*> v;
v.push_back(p); // 这样做是可以的
v[0] = new int(42);
cout << *v[0]; // 42
cout << *p; // 错误：p没有被初始化
```
* 上述是一个容易在Qt中出现的错误
```cpp
QVector<QLineEdit*> v { lineEdit1, lineEdit2 };
for(int i = 0; i < v.size(); ++i)
{
    v[i] = new lineEdit;
}
lineEdit1->setText("test"); // 错误，控件未被初始化
```

## vector扩容重新分配内存的过程
* [allocate](https://en.cppreference.com/w/cpp/memory/allocator/allocate)：分配未构造的新内存块（大小一般是原来的1.5倍或2倍）
```cpp
allocator<T> newAllocator;
auto p = newAllocator.allocate(v.size() * 2);
```
* [construct](https://en.cppreference.com/w/cpp/memory/allocator/construct)：把元素从容器的旧内存拷贝到新内存
```cpp
auto q = p;
for(int i = 0; i < v.size(); ++i)
{
    newAllocator.construct(q++, v[i]);
}
```
* [destroy](https://en.cppreference.com/w/cpp/memory/allocator/destroy)：调用~T()析构旧内存中的对象
```cpp
// p2指向oriAllocator首位置，q2指向尾后位置
while(q2 != p2) oriAllocator.destroy(--q2);
```
* [deallocate](https://en.cppreference.com/w/cpp/memory/allocator/deallocate)：释放旧内存
```cpp
oriAllocator.deallocate(p2, v.size());
```

## 使用实例
* 矩阵乘法
```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    int m = 2; // 矩阵1行数
    int n = 2; // 矩阵1列数和矩阵2行数
    int r = 2; // 矩阵2列数
    int temp;
    vector<int> row; // 用来临时保存行
    vector<vector<int>> matrix1;
    vector<vector<int>> matrix2;
    vector<vector<int>> result;
    // 矩阵1
    cout << "Input first(" << m << "*" << n << ")matrix:" << endl;
    for (int i = 1; i <= m; ++i)
    {
        cout << "Type in " << n << " values for row" << i
            << " separated by spaces:";
        for (int j = 1; j <= n; ++j)
        {
            cin >> temp;
            row.push_back(temp);
        }
        matrix1.push_back(row);
        row.clear();
    }
    // 矩阵2
    cout << "Input second(" << n << "*" << r << ")matrix:" << endl;
    for (int i = 1; i <= n; ++i)
    {
        cout << "Type in " << r << " values for row" << i
            << " separated by spaces:";
        for (int j = 1; j <= r; ++j)
        {
            cin >> temp;
            row.push_back(temp);
        }
        matrix2.push_back(row);
        row.clear();
    }
    // 计算结果
    for (int i = 0; i < m; ++i)
    {
        for (int j = 0; j < r; ++j)
        {
            temp = 0;
            for (int k = 0; k < n; ++k)
            {
                temp += matrix1[i][k] * matrix2[k][j];
            }
            row.push_back(temp);
        }
        result.push_back(row);
        row.clear();
    }
    // 打印结果
    for (auto x : matrix1)
    {
        for (auto y : x) cout << y << " ";
        cout << endl;
    }
    cout << "times" << endl;
    for (auto x : matrix2)
    {
        for (auto y : x) cout << y << " ";
        cout << endl;
    }
    cout << "equals" << endl;
    for (auto x : result)
    {
        for (auto y : x) cout << y << " ";
        cout << endl;
    }
}
```
* 大数阶乘
```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<char> result; // 也可以用std::string
    // result.reserve(10000);
    result.emplace_back(1); // 默认值为1
    int n = 100;
    for (int i = 1; i <= n; ++i) // 依次计算i的阶乘
    {
        int carry = 0; // 进位值
        for (std::size_t j = 0; j < result.size(); ++j) // 依次计算结果的各个数位
        {
            int tmp = result[j] * i + carry;
            result[j] = tmp % 10;
            carry = tmp / 10; // 其他用于进位
        }
        while (carry > 0) // 如果还有多余的进位值依次置于高位
        {
            result.emplace_back(carry % 10); // 放置高位值
            carry /= 10;
        }
    }
    std::cout << n << "! = ";
    for (auto it = result.crbegin(); it != result.crend(); ++it)
    {
        std::cout << static_cast<int>(*it);
    }
}
```
