---
title: "C++练习题"
date: 2023-06-18T14:05:25+08:00
tags: ["C++"]
categories: []
draft: false
---
## 多态

### 判断

- Q1: 虚函数可以是内联的？
  A1: 错误。内联是编译时刻决定的，虚函数是运行时刻动态决定的，所以虚函数不能是内联函数。虚函数前加上inline不会报错，但是会被忽略。

- Q2: 一个类内部，可以同时声明 `static void fun()` 和 `virutal void fun()` 两个函数？
  A2: 错误。虽然静态函数不存在this指针，但是还是不能声明同参的虚函数和静态函数。

- Q3: 基类的析构函数非虚，派生类的析构函数是虚函数。delete派生类指针(指向派生类对象)会调用基类析构函数？
  A3: 正确。

  > 通过**派生类指针**，删除派生类对象时，无论父类析构函数是不是虚函数，都会调用基类析构函数。
  >
  > 通过**基类指针**，删除派生类对象时，是否调用基类析构函数，取决于基类析构函数是否是virtual函数

### 知识点

可以通过对象名主动调用析构函数，主动调用构造函数会出错。

如果类中声明了构造函数（无论是否有参数），编译器便不会再为之生成隐含的构造函数。

构造函数不能是虚函数，析构函数可以是虚函数。 

### 编程

#### 析构函数需要声明为virtual

```cpp
#include <iostream>
using namespace std;

int g_num = 0;

class Base
{
public:
    Base() {}
    ~Base() { g_num += 1; }
};

class Derived:public Base
{
public:
    Derived() {}
    ~Derived() { g_num += 2; }
};

int main()
{
    Base *p = new Derived();
    delete p;
    cout << g_num << endl;
    return 0;
}
// 输出：1
```

#### 动态绑定依赖于指针或者引用

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun() {cout << "base" << endl;}
};

class Derived:public Base
{
public:
    virtual void fun() { cout << "derived" << endl;}
};

void func1(Base &obj) {
    obj.fun();
}
void func2(Base obj) {
    obj.fun();
}

int main()
{
    Base *pBase = new Derived();
    func1(*pBase); // derived
    func2(*pBase); // base
    return 0;
}
```

#### operator可以定义为虚函数

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void operator == (int val2) {
        cout << "base: " << val2 << endl;
    }
};

class Derived:public Base
{
public:
    virtual void operator == (int val2) {
        cout << "derived: " << val2 << endl;
    }
};

int main()
{
    Base *pBase = new Derived();
    Derived *pDerived = new Derived();
    *pBase == 10;
    *pDerived == 20;
}

// derived: 10
// derived: 20
```

#### 绝不重新定义继承而来的缺省参数

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun(string str = "base123") {
        cout << str << endl;
    }
};

class Derived:public Base
{
public:
    virtual void fun(string str = "derived") {
        cout << str << endl;
    }
};

int main()
{
    Base *pBase = new Derived();
    pBase->fun(); // base123
}
```

#### 同名const函数属于有效重载

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    void func1() {cout << "non-const\n";}
    void func1() const {cout << "const\n";}
};

int main()
{
    Base b1;
    b1.func1(); // non-const。非常量对象也可以调用const函数，此处优先调用non-const函数

    const Base b2;
    b2.func1(); // const。常量对象只能调用const函数
}
```

#### virtual函数的const必须统一

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void display() const { cout << "Base"; }
};

class Derived:public Base
{
public:
    virtual void display() { cout << "Derived"; }
};

int main()
{
    Base *pBase = new Derived();
    pBase->display(); // Base
}
```

#### 派生类生成过程

> 派生类构造函数执行顺序：
>
> 1.调用基类构造函数，调用顺序按照它们**被继承时声明的顺序**（从左向右）
>
> 2.对派生类新增的成员对象初始化，调用顺序按照它们**在类中声明的顺序**
>
> 3.执行派生类的构造函数体中的内容。

```cpp
#include <iostream>
#include <string>
using namespace std;

class A {
public:
    A(string name) {cout << name << " ";}
};

class Base {
public:
    Base(string str) : a1(str) {}
public:
    A a1;
};

class Derived : public Base {
public:
    Derived() : a4("a4"), Base("a1"), a2("a2") {}
public:
    A a2;
    A a3 = A("a3");
    A a4;
};

int main()
{
    Derived d;
    return 0;
}
// 输出：a1 a2 a3 a4
```

```cpp
#include <iostream>
#include <string>
using namespace std;

class Base {
public:
    Base(int val) : a(val) {}
protected:
    int a;
};

class Derived : public Base {
public:
    Derived(int val1, int val2) : c(val2), Base(val1), b(a + c) {
        cout << a << " " << b << " " << c << endl; // a==1, b的值不确定, c==10
    }
private:
    int b;
    int c;
};

int main()
{
    Derived d(1, 10);
}
```
