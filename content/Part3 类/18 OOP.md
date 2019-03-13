## 基类和派生类的定义
* 派生类必须通过类派生列表指出基类，类派生列表的形式是，首先是一个冒号，后面紧跟逗号分隔符的基类列表，每个基类前可以有访问说明符，表示继承方式，用来控制派生类用户（包括派生类的派生类）对基类成员的访问权限
```cpp
class Derived : public Base {
    ...
};
```
* 每个类控制它自己的成员初始化过程，**派生类必须使用基类的构造函数来初始化它的基类部分**，首先初始化基类的部分，然后按照声明顺序依次初始化派生类成员
```cpp
Derived(const std::string& s, double p, std::size_t n) :
    Base(s, p), num(n) {}
// 该函数将前两个参数传递给基类的构造函数来初始化基类部分
// 基类首先被初始化完成后，按照声明顺序依次初始化派生类成员
```
* C++11中，派生类可以重用其直接基类定义的构造函数。类不能继承默认、拷贝和移动构造函数
```cpp
class Derived : public Base {
public:
    using Base::Base; // 继承基类的构造函数
    ...
}

// 相当于省略了如下形式的定义
class Derived : public Base {
public:
    Derived(const std::string& s, double p) : Base(s, p) {}
    ...
}
```
* 不同于普通函数的using声明，构造函数的using声明不会改变访问级别。不管using声明出现在哪，基类的private构造函数在派生类中还是一个private构造函数，protected和public同理
* 基类构造函数含有默认实参时，这些实参不会被继承，相反，派生类将获得多个继承的构造函数，其中每个构造函数分别省略掉一个含有默认实参的形参。例如，如果一个基类有一个形参数为2的构造函数，第二个形参有默认实参，则派生类获得两个构造函数：一个构造函数接受两个形参（无默认实参），另一个只接受一个形参（对应基类第一个无默认实参的形参）
* 如果基类有几个构造函数，除以下两种情况，大多数时候派生类会继承所有这些构造函数
  * 第一个例外是派生类可以继承一部分构造函数，而为其他构造函数定义自己的版本。如果派生类定义的构造函数与基类构造函数的参数列表相同，则该构造函数不会被继承。定义在派生类中的构造函数将替换继承而来的构造函数
  * 第二个例外是默认、拷贝和移动构造函数不会被继承。这些构造函数按正常规则被合成。继承的构造函数不会被作为用户定义的构造函数来使用，因此如果一个类只含有继承的构造函数，则它也将拥有一个合成的默认构造函数
* 每个类负责定义各自的接口，要想与类的对象交互必须使用该类的接口，即使此对象是派生类的基类部分。语法上可以在派生类构造函数体内给基类赋值，但是为了遵循基类接口，要通过调用基类的构造函数来初始化
* 若基类中定义了static成员，则整个继承体系中只存在该成员的唯一定义，静态成员遵循通用的访问控制规则
```cpp
class Base {
public:
    static void f();
};
class Derived : public Base {
    void g(const Derived&);
};
void Derived::g(const Derived &derived_obj)
{
    Base::f(); // 正确，Base定义了f
    Derived::f(); // 正确，Derived继承了f
    derived_obj.f(); // 通过Derived对象访问
    f(); // 通过this对象访问
}
```
* 派生类的声明和其他类一样，包含类名，不包含派生列表
```cpp
class Derived : public Base; // 错误的声明
class Derived;
```
* **用作基类的类必须已经定义，这也表明一个类不能派生它本身**
```cpp
class Base; // 声明但未定义
class Derived : public Base { ... };  // 错误，基类未定义
class Derived : public Derived { .. }; // 错误，不能用类本身作为基类
```
* 一个类可以同时是基类和派生类
```cpp
class Base { /*...*/ }; // Base是D1的直接基类，是D2的间接基类
class D1 : public Base { /*...*/ }; // D1既是派生类又是基类
class D2 : public D1 { /*...*/ };
```
* **C++11提供在类名后加final关键字的方法来防止继承发生**
```cpp
class NoDerived final { /*...*/ }; // NoDerived不能作为基类
class Base { /*...*/ };
class Last final : Base { /*...*/ }; // Last不能作为基类
class Bad : NoDerived { /*...*/ }; // 错误
class Bad2 : Last { /*...*/ }; // 错误
```

## 派生类到基类的类型转换
* 因为派生类对象中含有基类对应的组成部分，因此可以把派生类对象当成基类对象使用，也可以将基类的指针或引用绑定到派生类对象的基类部分上，这种转换称为派生类到基类（derived-to-base）类型转换，**这种转换只对指针或引用类型有效**，此时使用基类的引用或指针并不知道绑定对象的真实类型，可能是基类对象，也可能是派生类
```cpp
Base b; // 基类对象
Derived d; // 派生类对象
Base* p = &b;
p = &d; // p指向d的Base部分
Base& r = d; // r绑定到d的Base部分
```
* 派生类到基类的转换只在public继承中有效
```cpp
class A{};
class B : public A{};
class C : protected A{};
class D : private A{};
void f(const A& x) {}

A* pb = new B; // OK
A* pc = new C; // 错误：转换存在但无法访问
A* pd = new D; // 错误：转换存在但无法访问

B b;
C c;
D d;
f(b); // OK
f(c); // 错误
f(d); // 错误
```
* 静态类型总是已知的，动态类型在运行时才可知，如果表达式不是指针或引用，它的动态类型和静态类型永远一致，如Base类型的变量永远是一个Base对象，该变量类型永远不可改，但指针或引用的静态类型可能和动态类型不一致
* **不存在基类到派生类的转换，即使基类指针或引用绑定到了派生类对象上也不能执行基类到派生类的转换**，因为编译器在编译时只能通过检查指针或引用的静态类型推断该转换是否合法，不能确定运行时该转换是否安全。但如果已知转换是安全的，可以通过static_cast强制覆盖编译器的检查工作，如果基类中有虚函数，可以用dynamic_cast，该转换的安全检查在运行时执行
```cpp
Base b;
Derived* pd = &b; // 错误 
Derived& rd = b; // 错误
// 如果上述赋值合法，它们可能访问b中不存在的成员
Derived d;
Base* pb = &d; // 合法
Derived* pd = pb; // 错误
```
* 用派生类对象给基类对象初始化或赋值时，实际上是调用基类的拷贝/移动控制，只有基类部分会被拷贝、移动或赋值，派生类部分被切掉

## 虚函数
* OOP的核心思想是多态性，具有继承关系的多个类型称为多态类型，引用或指针的静态类型与动态类型不同正是C++支持多态性的根本所在。使用基类的引用或指针调用基类定义的一个函数时，该函数的作用对象类型可能是基类对象也可能是派生类对象，如果该函数是虚函数，运行时才会决定执行哪个版本
* **任何构造函数之外的非静态函数都可以是虚函数，一旦某个函数被声明成虚函数，所有派生类中无论是否加virtual都是虚函数，为方便阅读通常加virtual**
* 调用虚函数时只有在运行时才知道调用了哪个版本，因此**虚函数必须有定义，不管它是否被用到**（纯虚函数则不需要）
* **派生类的函数如果覆盖了某个继承而来的虚函数，则形参类型和返回类型与被它覆盖的基类函数一致**，但返回类型匹配存在一个例外：返回类型是类本身的指针或引用。如基类虚函数返回B\*而派生类返回D\*，不过这样的返回类型要求D到B的转换是可访问的
* **如果形参列表不同，虽然合法，但编译器会认为这个函数与基类中的函数是相互独立的，并没有重写基类版本，这样实际上违背了本意并难以发现**，C++11使用override关键字标记派生类中的虚函数，被override标记的函数如果没有重写已存在的虚函数，编译器会报错
```cpp
struct B {
    virtual void f1(int) const;
    virtual void f2();
    void f3();
};
struct D1 : B {
    void f1(int) const override; // 正确，与基类中的f1匹配
    void f2(int) override; // 错误，基类没有形如f2(int)的函数
    void f3() override; // 错误，f3不是虚函数，只有虚函数才能被重写
    void f4() override; // 错误，基类没有名为f4的函数
};
```
* 也可以像用final防止继承那样将函数定义为final，之后任何尝试重写该函数的操作都会引发错误
```cpp
struct D2 : B {
    // 从B继承f2()和f3()，重写f1(int)
    void f1(int) const final;
};
struct D3 : D2 {
    void f2(); // 正确，重写从间接基类B继承而来的f2
    void f1(int) const; // 错误，D2已经将f1声明了final
};
```
* **虚函数也可以有默认实参，调用时实参值由本次调用的静态类型决定，如果通过基类的引用或指针调用函数则使用基类定义的默认实参，即使实际运行的是派生类的版本，此时传入派生类函数的是基类定义的默认实参，如果派生类函数依赖不同的实参程序结果将与预期不符，因此要使用默认实参，基类和派生类中定义的默认实参最好一致**
```cpp
class Base {
public:
    virtual void f(int i = 10);
};
void Base::f(int i)
{
    cout << i << endl;
}

class Derived : public Base {
public:
    void f(int i = 20);
};
void Derived::f(int i)
{
    cout << "Derived::f() " << i << endl;
}

int main()
{
    Base* pb = new Derived;
    pb->f(); // 打印Derived::f() 10
}
```
* 如果希望对虚函数的调用不进行动态绑定，而是强迫执行某个特定版本，使用作用域运算符实现目的。通常只有成员函数或友元才需要使用它来回避虚函数的机制，如一个派生类的虚函数调用覆盖的基类的虚函数版本
* **基类指针指向派生类对象时，调用普通方法会调用基类的方法，调用虚函数时会调用派生类重写的虚函数方法，如果没有重写就往上调用基类的虚函数**
* **想在基类中抽象出一个方法，该基类只能被继承但不能实例化，使用纯虚函数来实现，纯虚函数无须定义。用=0将一个虚函数说明为纯虚函数，=0只能出现在类内部的虚函数声明处**
* 含有或未经重写直接继承纯虚函数的类是抽象基类，不能创建抽象基类的对象，抽象基类负责定义接口，后续其他类可以重写该接口
* 含有虚函数的类都有一张虚函数表（vtbl），一个vtbl通常是一个函数指针数组（一些编译器用链表），一个类的vtbl的大小与类中声明的虚函数数量成正比（包括从基类继承而来的虚函数）。每个类应该只有一个vtbl，所以vtbl所需空间不会太大，但如果有大量类或每个类有大量虚函数，vtbl就会占用大量地址空间
* vtbl只实现了虚函数一半机制，另一半靠vptr实现，vptr的作用是指出每个对象对应的vtbl。每个声明了虚函数的对象都带有vptr，vptr是一个看不见的数据成员，被编译器加在对象里，位置只有编译器知道。如果对象很小，vptr的代价就很大，比如对象平均只有四个字节，vptr就会使成员数据大小增加一倍
* **继承关系要求基类定义一个虚析构函数，delete指针时执行析构函数，如果指针指向继承体系中的某个类型，可能指针的静态类型与被删除的动态类型不符，此时需要虚函数来确保delete时选择正确的析构函数版本。如delete一个基类指针，该指针可能指向派生类对象，此时编译器必须知道应该执行派生类的析构函数。如果基类的析构函数不是虚函数，delete指向派生类对象的基类指针会产生未定义行为。和其他虚函数一样，析构函数的虚属性也会被继承。根据三/五法则，如果一个类需要析构函数，则同样需要拷贝和赋值操作，但基类的析构函数例外。虚析构函数会阻止合成移动操作，即使通过=default使用合成版本**
```cpp
class A {
public:
    A() { cout << "create A\n"; }
    ~A() { cout << "delete A\n"; }
};
class B : public A {
public:
    B() { cout << "create B\n"; }
    ~B() { cout << "delete B\n"; }
    int x;
};
int main()
{
    A *a = new B;
    delete a;
}
```
* 基类中没有把析构函数声明为虚函数，因此删除基类指针不会调用B的析构函数，x没被释放导致内存泄漏

![](http://upload-images.jianshu.io/upload_images/5587614-62ea0c51bc3807be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 将A和B的析构函数声明为虚析构函数将会调用B的析构

![](http://upload-images.jianshu.io/upload_images/5587614-14db7ba883315281.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##访问控制与继承
* private只能被该类函数或友元函数访问，不能被其他包括该类对象访问
* protected可以被该类函数或友元函数和派生类函数访问
* public可以被该类函数、友元函数、子类函数、该类对象访问
* 继承后的访问权限变化

|继承方式\基类成员|B::public|B::protected|B::private|
|:-:|:-:|:-:|:-:|
|D : public B|D::public|D::protected|B::private|
|D : protected B|D::protected|D::protected|B::private|
|D : private B|D::private|D::private|B::private|
* 派生类的成员和友元只能访问派生类对象中基类部分的proteced成员，即**只能通过派生类对象而不是基类对象来访问基类的受保护成员**
```cpp
class Base {
protected:
    int prot_mem;
};
class Sneaky : public Base {
    friend void clobber(Sneaky&); // 能访问Sneaky::prot_mem
    friend void clobber(Base&); // 不能访问Base::prot_mem
    int j; // j默认为private
};
void clobber(Sneaky &s) { s.j = s.prot_mem = 0; } // 正确
void clobber(Base &b) { b.prot_mem = 0; } // 错误
```
* 派生类向基类转换的可访问性
  * 如果是公有继承（class D : public B），用户代码可以使用D向B的转换，否则用户代码不能使用该转换
  * 不论D怎么继承B，D的成员函数和友元都能使用D向B的转换
  * 如果是public或protected继承，D的派生类成员和友元可以使用D向B的转换
* **友元不能传递，也不能继承**
* 使用using改变成员的可访问性
```cpp
public:
    using Base::size;
protected:
    using Base::n;
```
* 派生类的作用域嵌套在其基类的作用域之内，如果一个名字在派生类的作用域内无法正确解析，则编译器继续在外层的基类作用域寻找该名字的定义，如果名字冲突，派生类的成员将隐藏同名的基类成员，可以通过作用域运算符使用被隐藏的基类成员
* **声明在内层作用域的函数不会重载而是隐藏声明在外层作用域的函数，因此定义在派生类中的函数不会重载基类中的成员**，如果派生类成员与基类某个成员同名，即使形参列表不一致，基类成员也会被隐藏。由此可以理解基类和派生类的虚函数要有相同形参列表的原因，如果接受的实参不同，就不能通过基类的引用或指针调用派生类的虚函数
* **基类指针指向派生类对象时，调用非虚函数则调用基类方法，调用虚函数时会调用派生类重写的虚函数，如果没有重写就往上调用基类的虚函数**，基类指针指向派生类再调用派生类重写的虚函数就是多态
```cpp
class Base {
public:
    virtual int f();
};
chass D1 : public Base {
public:
    int f(int); // 隐藏了基类的f，此f(int)不是虚函数
    virtual void f2(); // 新的虚函数
};
class D2 : public D1 {
public:
    int f(int); // 非虚函数，隐藏了D1::f(int)
    int f(); // 覆盖Base的虚函数f()
    void f2(); // 覆盖D1的虚函数f2
};

Base b;
D1 d1;
D2 d2;

Base* bp1 = &b;
Base* bp2 = &d1;
Base* bp3 = &d2;
// 编译器在运行时确定虚函数版本，判断依据是该指针绑定对象的真实类型
bp1->f(); // 运行时调用Base::f()
bp2->f(); // 运行时本来调用D1::f()，但D1没有这个虚函数，往上调用Base::f()
bp2->f2(); // 错误，Base没有f2()
bp3->f(); // 运行时调用D2::f()

D1* d1p = &d1;
D2* d2p = &d2;
d1p->f2(); // 运行时调用D1::f2()
d2p->f2(); // 运行时调用D2::f2()

// 再看看对非虚函数f(int)的调用
Base* p1 = &d2;
D1* p2 = &d2;
D2* p3 = &d2;
p1->f(42); // 错误，Base没有f(int)
p2->f(42); // 静态绑定，调用D1::f(int)
p3->f(42); // 静态绑定，调用D2::f(int)
```
* 再举一个例子
```cpp
struct A
{
    A() : x(0) {}
    virtual void foo() {
        cout << "A: " << x++ << endl;
    }
    void bar() {
        cout << "A: " << (x+=5) << endl;
    }
    int x;
};
struct B : A
{
    B() : y(10) {}
    void foo() {
        cout << "B:"<< ++y << endl;
        A::foo();
    }
    void bar() {
        cout << "B: " << (y-=3) << endl;
        A::bar();
    }
    int y;
};
int main()
{
    B b; // x = 0，y = 10
    A& a(b);
    a.bar(); // 调用A::bar()，x += 5
    // A : 5
    // 此时 x = 5
    b.bar(); // 调用B::bar()，y -= 3
    // B : 7
    // 此时 y = 7
    // 再调用A::bar()，x += 5
    // A : 10
    // 此时 y = 7，x = 10
    a.foo(); // 调用B::foo()，++y
    // B : 8
    // 再调用A::foo()，x++
    // A : 10
    // 此时 y = 8，x = 11
    b.foo(); // 调用B::foo()，++y
    // B : 9
    // 再调用A::foo()，x++
    // A : 11
    // 此时 y = 9，x = 12
}

// 结果如下
// a.bar()
A : 5
// b.bar()
B : 7
A : 10
// a.foo()
B : 8
A : 10
// b.foo()
B : 9
A : 11
```

##多重继承与虚继承
* 多重继承的情况下，如果名字在多个基类中被找到，而当前派生类没有覆盖此名字，或没有明确指出调用版本，则对该名字的使用具有二义性
```cpp
struct A { int i = 3; };
struct B { int i = 4; };
struct C : public A, public B {};
struct D : public A, public B { int i = 5; };
C c;
D d;
cout << c.i; // 二义性错误，无法确定访问的是A::i还是B::i
cout << c.A::i; // 3
cout << c.B::i; // 4
cout << d.i; // 5
```
* 注意，间接基类中找到也一样
```cpp
struct A { int i = 3; };
struct B { int i = 4; };
struct C : public B {};
struct D : public A, public C {}; // A是直接基类，B是间接基类
D d;
cout << d.i; // 二义性错误，i可能来自A或B
cout << d.A::i; // 3 
cout << d.B::i; // 4
cout << d.C::i; // 4
```
* 即使来自同一个间接基类仍然属于二义性错误
```cpp
struct A { int i = 3; };
struct B : public A {};
struct C : public A {};
struct D : public B, public C {};
D d;
cout << d.i; // 二义性错误
cout << d.A::i; // 二义性错误
cout << d.B::i; // 3
cout << d.C::i; // 3
```
* 因为D继承了两次A，D对象将包含A的多个子对象
```cpp
struct A { A() { cout << "a"; } };
struct B : public A { B() { cout << "b"; } };
struct C : public A { C() { cout << "c"; } };
struct D : public B, public C { D() { cout << "d"; } };
D d; // abacd
```
* 使用虚继承可以解决此问题
```cpp
// virtual和public位置随意，将A定义为B和C的虚基类
struct A { A() { cout << "a"; } };
struct B : virtual public A { B() { cout << "b"; } };
struct C : public virtual A { C() { cout << "c"; } };
struct D : public B, public C { D() { cout << "d"; } };
D d; // abcd
// D通过B和C继承了A，因为B和C都是虚继承自A，所以D中只有一个A基类部分
```
* 此时如果A定义了一个成员i，其在D的作用域中通过D的两个基类都是可见的，如果通过D对象使用i
  * 若B和C中都没有i的定义，i被解析为A成员，不存在二义性，一个D对象只含有一个A实例
  ```cpp
  struct A { int i = 3; };
  struct B : virtual public A {};
  struct C : virtual public A {};
  struct D : public B, public C {};
  D d;
  cout << d.i; // 3
  cout << d.A::i; // 3
  ```
  * 若i是B或C中的某一个成员，同样没有二义性，派生类的i比共享虚基类A的i优先级高
  ```cpp
  struct A { int i = 3; };
  struct B : virtual public A { int i = 4; };
  struct C : virtual public A {};
  struct D : public B, public C {};
  D d;
  cout << d.i; // 4
  ```
  * B和C中都有i的定义，直接访问i产生二义性，最好的解决方法是在D中定义新的i
  ```cpp
  struct A { int i = 3; };
  struct B : virtual public A { int i = 4; };
  struct C : virtual public A { int i = 5; };
  struct D : public B, public C {};
  D d;
  cout << d.i; // 二义性错误
  ```
* 虚继承中，虚基类由最低层的派生类初始化，创建D时，D的构造函数独自控制A的初始化过程，因为如果应用普通规则，虚基类A会被B和C重复初始化。继承体系中的每个类都可能在某个时刻成为最低层的派生类，比如创建B对象时，B对象就位于派生的最低层，因此B的构造函数将直接初始化A的基类部分
* 含有虚基类的对象的构造顺序与一般的顺序不同：先用最低层派生类的构造函数的初始值初始化虚基类子部分，然后按直接基类在派生列表中出现的顺序依次初始化
```
struct A { ... };
struct B : virtual public A { ... };
struct C : virtual public A { ... };
struct X { ... };
struct D : public B, public C, public X { int i; };
```
* 对上述继承关系，创建一个D对象时，顺序为
  * 用D的构造函数初始值列表中提供的初始值构造虚基类A部分
  * 构造B部分
  * 构造C部分
  * 构造直接基类X（虚基类总是先于非虚基类构造，与继承中的次序无关）
  * 构造D部分
* 因此D的构造函数初值列表按顺序写为
```cpp
D::D(...) : A(...), B(...), C(...), X(...), i(...) {}
```
* 如果D没有显式初始化A，则A的默认构造函数将被调用，如果A没有默认构造函数则出错
* 一个类可以有多个虚基类，此时虚的子对象按它们在派生列表中的顺序从左向右依次构造
```cpp
struct A { A() { cout << "a";} };
struct B : public A { B() { cout << "b";} };
struct C { C() { cout << "c";} };
struct D : public C { D() { cout << "d";} };
struct E { E() { cout << "e";} };
struct F : virtual public E { F() {cout << "f";} };
struct X : public B, public F, virtual public D { X() { cout << "x"; } };

X x; // ecdabfx
```
* F有两个虚基类：间接虚基类E和直接虚基类D。编译器按直接基类的声明顺序依次检查，确定其中是否含有虚基类，如果有先构造虚基类，然后按声明顺序构造其他非虚基类，因此创建一个F对象，顺序为
```cpp
E() // 直接基类F的虚基类
C() // 直接虚基类D的基类
D() // 直接虚基类
A() // 第一个非虚基类B的基类
B() // 第一个直接非虚基类B
F() // 第二个直接非虚基类F
X() // 最低层的派生类
```
* 合成的拷贝、赋值和移动构造函数顺序同上，销毁顺序则恰好相反
* 在VS中查看类的内存布局：`Properties - Configuration Properties - C/C++ - Command Line -
 Addtion Options - /d1 reportAllClassLayout`，查看单个类如类A的内存布局则添加`/d1 reportSingleClassLayoutA`，`Build - Rebuild Solution`后在`Output`中即可看到结果
```cpp
class A {
    virtual void f() = 0;
    virtual void g() {}
    virtual void h() {}
    int i;
    char a;
    double d;
};

class B : public A {
    virtual void g() {}
    virtual void f() {}
    int* p;
};
```

![](https://upload-images.jianshu.io/upload_images/5587614-9ddbedad5e8e8738.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 在Linux中查看类的内存布局：直接用下面的命令即可将xxx.cpp的内存布局导出到一个生成的xxx.cpp.002t.class文件中
```cpp
g++ -fdump-class-hierarchy xxx.cpp
```

![](https://upload-images.jianshu.io/upload_images/5587614-ca73614356706468.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
