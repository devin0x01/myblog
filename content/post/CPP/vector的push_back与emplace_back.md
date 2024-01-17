---
title: "vector的push_back与emplace_back.md"
date: 2023-07-17T14:05:25+08:00
tags: ["C++"]
categories: []
draft: false
toc: true
---
# push_back与emplace_back

```cpp
#include <iostream>
#include <vector>
using namespace std;

class A {
public:
    A() = default;
    A(string name) : _name(name) { cout << "ctor: " << _name << endl; }
    ~A() { cout << "dtor: " << _name << endl; }
    // 拷贝构造函数
    A(const A &other) { _name += "cp-" + other._name; cout << "cp-ctor: " << _name << endl; }
    // 赋值运算符
    A& operator=(const A &other) {
        if (this != &other) {
            _name += "cp-" + other._name;
            cout << "cp-op: " << _name << endl;
        }
        return *this;
    }
    // 移动构造函数
    A(A &&other) noexcept { _name += "mv-" + other._name; cout << "mv-ctor: " << _name << endl; }
    // 移动赋值运算符
    A& operator=(A &&other) noexcept {
        if (this != &other) {
            _name += "mv-" + other._name;
            cout << "mv-op: " << _name << endl;
        }
        return *this;
    }

    string _name;
};

int main()
{
    vector<A> arr;
    arr.reserve(100); // 重要！重新分配内存时可能导致测试结果有迷惑性

    printf("==== push_back(\"a1\") ====\n");
    arr.push_back(string("a1"));
    printf("\n==== emplace_back(\"a2\") ====\n");
    arr.emplace_back(string("a2"));

    printf("\n==== push_back/emplace_back(lvalue) ====\n");
    A a3("a3");
    arr.push_back(a3);
    arr.emplace_back(a3);

    printf("\n==== push_back/emplace_back(rvalue) ====\n");
    A a4("a4");
    arr.push_back(std::move(a4));
    A a5("a5");
    arr.emplace_back(std::move(a5));

    printf("\n==== return ====\n");
    return 0;
}
```

输出为:
```txt
==== push_back("a1") ====
ctor: a1
mv-ctor: mv-a1
dtor: a1

==== emplace_back("a2") ====
ctor: a2

==== push_back/emplace_back(lvalue) ====
ctor: a3
cp-ctor: cp-a3
cp-ctor: cp-a3

==== push_back/emplace_back(rvalue) ====
ctor: a4
mv-ctor: mv-a4
ctor: a5
mv-ctor: mv-a5

==== return ====
dtor: a5
dtor: a4
dtor: a3
dtor: mv-a1
dtor: a2
dtor: cp-a3
dtor: cp-a3
dtor: mv-a4
dtor: mv-a5
```

# 移动构造函数用noexcept修饰后，vector扩容时才会调用
[为什么需要将移动构造函数和移动赋值运算符标记为noexcept](https://young-flash.github.io/2021/12/18/%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81%E5%B0%86%E7%A7%BB%E5%8A%A8%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0%E5%92%8C%E7%A7%BB%E5%8A%A8%E8%B5%8B%E5%80%BC%E8%BF%90%E7%AE%97%E7%AC%A6%E6%A0%87%E8%AE%B0%E4%B8%BA%20noexcept/)

```cpp
#include <iostream>
#include <vector>
using namespace std;

class A {
public:
    A() = default;
    A(string name) : _name(name) { cout << "ctor: " << _name << endl; }
    ~A() { cout << "dtor: " << _name << endl; }
    // 拷贝构造函数
    A(const A &other) { _name = "cp-" + other._name; cout << "cp-ctor: " << _name << endl; }
    // 赋值运算符
    A& operator=(const A &other) {
        if (this != &other) {
            _name = "cp-" + other._name;
            cout << "cp-op: " << _name << endl;
        }
        return *this;
    }
    // 移动构造函数
    A(A &&other) noexcept { _name = "mv-" + other._name; cout << "mv-ctor: " << _name << endl; }
    // 移动赋值运算符
    A& operator=(A &&other) noexcept {
        if (this != &other) {
            _name = "mv-" + other._name;
            cout << "mv-op: " << _name << endl;
        }
        return *this;
    }

    string _name = "anonymous";
};

int main() {
    vector<A> arr(4);
    arr.reserve(4);
    cout << "capacity: " << arr.capacity() << endl; // 4

    A a1("a1");
    A a2("a2");
    arr.push_back(a1); // 触发vector内存的重新分配: 第5个元素调用cp-ctor
                       // 如果mv-ctor声明为noexcept，则前4个元素调用移动构造函数; 否则调用拷贝构造函数cp-ctor
    arr.push_back(std::move(a2)); // 第6个元素调用mv-ctor
    cout << "capacity: " << arr.capacity() << endl; // 8

    cout << "==== return ====" << endl;
}
```
输出信息为:
```txt
capacity: 4
ctor: a1
ctor: a2
cp-ctor: cp-a1
mv-ctor: mv-anonymous
dtor: anonymous
mv-ctor: mv-anonymous
dtor: anonymous
mv-ctor: mv-anonymous
dtor: anonymous
mv-ctor: mv-anonymous
dtor: anonymous
mv-ctor: mv-a2
capacity: 8
==== return ====
dtor: a2
dtor: a1
dtor: mv-anonymous
dtor: mv-anonymous
dtor: mv-anonymous
dtor: mv-anonymous
dtor: cp-a1
dtor: mv-a2
```