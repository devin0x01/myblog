---
title: "target_link_libraries中的PRIVATE INTERFACE PUBLIC.md"
date: 2023-07-24T18:05:25+08:00
tags: ["CMake"]
categories: []
draft: false
---
[CMake target\_link\_libraries Interface Dependencies \- Stack Overflow](https://stackoverflow.com/questions/26037954/cmake-target-link-libraries-interface-dependencies)
[CMake的链接选项：PRIVATE，INTERFACE，PUBLIC \- 知乎](https://zhuanlan.zhihu.com/p/493493849)

> If you are creating a shared library and your source cpp files #include the headers of another library (Say, QtNetwork for example), but your header files don't include QtNetwork headers, then QtNetwork is a `PRIVATE` dependency.
> 
> If your source files and your headers include the headers of another library, then it is a `PUBLIC` dependency.
>
> If your header files other than your source files include the headers of another library, then it is an `INTERFACE` dependency.
>
> Other build properties of `PUBLIC` and `INTERFACE` dependencies are propagated to consuming libraries.

# 测试PUBLIC
```cpp
// foo.h
int foo();

struct A {
	int num1;
	double num2;
};

// foo.cpp
#include "foo.h"

int foo()
{
	return sizeof(A);
}

// bar.h
#include "foo.h"

int bar();

// bar.cpp
#include "foo.h"

int bar()
{
	return foo() * 10;
}

// main.cpp
#include <iostream>
#include "bar.h"

int main()
{
	std::cout << foo() << std::endl;
	std::cout << bar() << std::endl;
	return 0;
}

// CMakeLists.txt
cmake_minimum_required(VERSION 3.16)

project(foobar)
set(CMAKE_SKIP_RPATH TRUE)

include_directories(${CMAKE_SOURCE_DIR})

add_library(foo SHARED foo.cpp)

add_library(bar SHARED bar.cpp)
target_link_libraries(bar PUBLIC foo)

add_executable(main main.cpp)
target_link_libraries(main bar)
```

```shell
$ export LD_LIBRARY_PATH=$PWD

$ ldd *.so
libbar.so:
	libfoo.so => /usr1/tmp/cmake1/build/libfoo.so (0x00007f8657389000)
libfoo.so:
	statically linked

$ ldd main
	libbar.so => /usr1/tmp/cmake1/build/libbar.so (0x00007f68298cc000)
	libfoo.so => /usr1/tmp/cmake1/build/libfoo.so (0x00007f68298c7000)

$ nm -C libbar.so | grep "foo\|bar"
0000000000001119 T bar()
                 U foo()

$ nm -C main | grep "foo\|bar"
                 U bar()
                 U foo()
```

# 测试PRIVATE
```cpp
// foo.h
struct A {
	int num1;
	double num2;
};

int foo();

// foo.cpp
#include "foo.h"

int foo()
{
	return sizeof(A);
}

// bar.h
int bar();

// bar.cpp
#include "foo.h"

int bar()
{
	return foo() * 10;
}

// main.cpp
#include <iostream>
#include "bar.h"

int main()
{
	std::cout << bar() << std::endl;
	return 0;
}

// CMakeLists.txt
cmake_minimum_required(VERSION 3.16)

project(foobar)
set(CMAKE_SKIP_RPATH TRUE)

include_directories(${CMAKE_SOURCE_DIR})

add_library(foo SHARED foo.cpp)

add_library(bar SHARED bar.cpp)
target_link_libraries(bar PRIVATE foo)

add_executable(main main.cpp)
target_link_libraries(main bar)
```

```shell
$ export LD_LIBRARY_PATH=$PWD

$ ldd *.so
libbar.so:
	libfoo.so => /usr1/tmp/cmake2/build/libfoo.so (0x00007fcaaa295000)
libfoo.so:
	statically linked

$ ldd main
	libbar.so => /usr1/tmp/cmake2/build/libbar.so (0x00007fea1beff000)
	libfoo.so => /usr1/tmp/cmake2/build/libfoo.so (0x00007fea1baa0000)

$ nm -C libbar.so | grep "foo\|bar"
0000000000001119 T bar()
                 U foo()

$ nm -C main | grep "foo\|bar"
                 U bar()
```

# 测试INTERFACE
```cpp
// foo.h
struct A {
	int num1;
	double num2;
};

int foo();

// foo.cpp
#include "foo.h"

int foo()
{
	return sizeof(A);
}

// bar.h
#include "foo.h"

int bar();

// bar.cpp
int bar()
{
	return 123 * 10;
}

// main.cpp
#include <iostream>
#include "bar.h"

int main()
{
	std::cout << foo() << std::endl;
	std::cout << bar() << std::endl;
	return 0;
}

// CMakeLists.txt
cmake_minimum_required(VERSION 3.16)

project(foobar)
set(CMAKE_SKIP_RPATH TRUE)

include_directories(${CMAKE_SOURCE_DIR})

add_library(foo SHARED foo.cpp)

add_library(bar SHARED bar.cpp)
target_link_libraries(bar INTERFACE foo)

add_executable(main main.cpp)
target_link_libraries(main bar)
```

```shell
$ export LD_LIBRARY_PATH=$PWD

$ ldd *.so
libbar.so:
	statically linked
libfoo.so:
	statically linked

$ ldd main
	libbar.so => /usr1/tmp/cmake3/build/libbar.so (0x00007f80dd28e000)
	libfoo.so => /usr1/tmp/cmake3/build/libfoo.so (0x00007f80dd289000)

$ nm -C libbar.so | grep "foo\|bar"
00000000000010f9 T bar()

$ nm -C main | grep "foo\|bar"
                 U bar()
                 U foo()
```