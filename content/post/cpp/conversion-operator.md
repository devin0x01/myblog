---
title: "conversion-operator"
date: 2023-08-14T16:05:25+08:00
tags: ["C++"]
categories: []
draft: false
toc: true
---

# 参考文档
[user-defined conversion function - cppreference.com](https://en.cppreference.com/w/cpp/language/cast_operator)  
[The Safe Bool Idiom - 知乎](https://zhuanlan.zhihu.com/p/30173442)  

一般形式为`operator type() const`, 比如:
```
operator int() const;
operator bool() const;
operator AA() const;
```

## 自定义类型转换
```cpp
struct To
{
    To() = default;
    To(const struct From&) {} // converting constructor
};

struct From
{
    operator To() const {return To();} // conversion function
};

int main()
{
    From f;
    To t1(f);  // direct-initialization: calls the constructor
    // Note: if converting constructor is not available, implicit copy constructor
    // will be selected, and conversion function will be called to prepare its argument

//  To t2 = f; // copy-initialization: ambiguous
    // Note: if conversion function is from a non-const type, e.g.
    // From::operator To();, it will be selected instead of the ctor in this case

    To t3 = static_cast<To>(f); // direct-initialization: calls the constructor
    const To& r = f;            // reference-initialization: ambiguous
}
```


## 为什么`operator bool()`需要用explicit修饰? 
[c++ - Why does declaring an `operator bool() const` member overload the [] operator? - Stack Overflow](https://stackoverflow.com/questions/76144901/why-does-declaring-an-operator-bool-const-member-overload-the-operator)  

The operator is coming from the built-in subscript operator which treats expressions `A[B]` as `*(A + B)`.  
This results in the evaluation of `*(1 + "wut")` => `'u'`, which then causes the `if` condition to pass, as `'u'` is a non-zero value.  
Declare your member as explicit operator bool() to prevent your type from being implicitly converted to other integral types.  

```cpp
#include <iostream>
using namespace std;

struct Test {
    operator bool() const {
        return true;
    }
};

int main(int argc, char** argv) {
    Test test;

    if (test)
        cout << "Object test is true\n";

    if (test["wut"])
        std::cout << "Success (test[\"wut\"])\n";
}

// Output:
// Object test is true
// Success (test["wut"])
```

## 一个`operator bool()`的坑
[c++ - Why is my "explicit operator bool()" not called? - Stack Overflow](https://stackoverflow.com/questions/24489762/why-is-my-explicit-operator-bool-not-called)  
