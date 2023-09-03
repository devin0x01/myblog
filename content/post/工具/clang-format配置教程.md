---
title: "clang-format配置教程"
date: 2023-08-29T16:05:25+08:00
tags: ["clang"]
categories: []
draft: false
---

# 配置clang-format
[Qt Creator使用clang-format_利白的博客-CSDN博客](https://blog.csdn.net/libaineu2004/article/details/104985934)  
[Git 如何将clang-formatting添加到预提交钩子|极客教程](https://geek-docs.com/git/git-questions/12_git_how_do_i_add_clangformatting_to_precommit_hook.html)  

clang-format二进制文件下载：https://llvm.org/builds  
Qt Creator自定义clang-format配置文件位置：`C:\Users\<username>\AppData\Roaming\QtProject\qtcreator\beautifier\clangformat`

# clang-format选项
[Clang-Format Style Options — Clang 18.0.0git documentation](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)  
.clang-format配置文件是yaml语法格式。  
**注意：配置文件要保存为UTF-8编码，否则可能会格式化代码失败。**  
```yaml
# 语言
Language: Cpp
# 基础样式
BasedOnStyle: LLVM
# 允许修改头文件顺序
SortIncludes: false
# 指针的*的挨着哪边,例如: int* ptr
DerivePointerAlignment: true
PointerAlignment: Left
# 访问修饰符前的空格,例如: public/private
AccessModifierOffset: -4
# 缩进宽度
IndentWidth: 4
# 要保留的最大连续空行数
MaxEmptyLinesToKeep: 2
# 大括号{}的换行方式，也可以定义为Custom，然后对if/class等分别设置
BreakBeforeBraces: Linux
# 是否允许短方法单行,例如: int f() { return 0; }
AllowShortFunctionsOnASingleLine: true
# 支持一行的if表达式，例如: if (a) return;
AllowShortIfStatementsOnASingleLine: false
# 在未封闭(括号的开始和结束不在同一行)的括号中的代码是否对齐,为true,则将参数在左方括号后水平对齐
AlignAfterOpenBracket: true
# switch的case缩进
IndentCaseLabels: true
# 每行字符的长度
ColumnLimit: 120
# 注释对齐
AlignTrailingComments: true
# 括号后加空格,例如: (int) i;
SpaceAfterCStyleCast: false
# 换行的时候对齐操作符
AlignOperands: true
# 中括号两边空格 []
SpacesInSquareBrackets: false
# 多行声明语句按照=对齐
AlignConsecutiveDeclarations: false
# 容器类的空格 例如: OC的字典
SpacesInContainerLiterals: false
# 构造函数初始化列表，冒号后面断行
BreakConstructorInitializers: AfterColon
# 函数参数换行
AllowAllParametersOfDeclarationOnNextLine: true
# 在续行(#下一行)时的缩进长度
ContinuationIndentWidth: 4
# tab键盘的宽度
TabWidth: 4
# 赋值运算符前加空格
SpaceBeforeAssignmentOperators: true
# 行尾的注释前加1个空格
SpacesBeforeTrailingComments: 1
```