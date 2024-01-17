---
title: "VimScript笔记"
date: 2024-01-17T15:05:25+08:00
tags: ["vim"]
categories: []
draft: false
toc: true
---

[VimScript 五分钟入门（翻译） - 知乎](https://zhuanlan.zhihu.com/p/37352209)
[wsdjeg/vim-plugin-dev-guide: Vim 插件开发指南](https://github.com/wsdjeg/vim-plugin-dev-guide)

# 基本语法
`:source %`: %表示当前文件的路径  
`e #`: 切换到最近编辑的另一个文件

expand() 将具有特殊意义的标记（如%，#，<cword> 等）展开

## 文件名修饰

文件名修饰是指如何从一个文件名中获取其目录、全路径名、后缀名等相关的名字字符串。函数 fnamemodify({fname}, {mods}) 的第二参数就叫做修饰符，修饰符以冒号开头带一个单字母表示不同意义，且可连续使用。主要的修饰符如：
- `:p` 文件全路径名
- `:h` 父目录名（文件名头部，去除路径分隔符最后一部分）
- `:t` 文件名尾部（一般是 ':h' 剩余部分，纯文件名）
- `:e` 文件名后缀
- `:r` 文件名主体（相对于 ':e' 而言，不包括后缀，但可能包含父目录）
## 变量
- `let` 命令用来对变量进行初始化或者赋值。
- `unlet` 命令用来删除一个变量。
- `unlet!` 命令同样可以用来删除变量，但是会忽略诸如变量不存在的错误提示。

默认情况下，如果一个变量在函数体以外初始化的，那么它的作用域是全局变量；而如果它是在函数体以内初始化的，那它的作用于是局部变量。同时你可以通过变量名称前加冒号前缀明确的指明变量的作用域：

```text
g:var - 全局
a:var - 函数参数
l:var - 函数局部变量
b:var - buffer 局部变量
w:var - window 局部变量
t:var - tab 局部变量
s:var - 当前脚本内可见的局部变量
v:var - Vim 预定义的内部变量
```
## 字符串比较
- `<string>` == `<string>`: 字符串相等
- `<string>` != `<string>`: 字符串不等
- `<string>` =~ `<pattern>`: 匹配 pattern
- `<string>` !~ `<pattern>`: 不匹配 pattern
- `<operator>#`: 匹配大小写
- `<operator>?`: 不匹配大小写

注意：设置选项 `ignorecase` 会影响 == 和 != 的默认比较结果，可以在比较符号添加 ? 或者 # 来明确指定大小写是否忽略。
## 函数
强制创建一个全局函数（使用感叹号），参数使用 `...` 这种不定长的参数形式时，a:1 表示 `...` 部分的第一个参数，a:2 表示第二个，如此类推，a:0 用来表示 `...` 部分一共有多少个参数。
```vim
function! g:Foobar(arg1, arg2, ...)
    let first_argument = a:arg1
    let index = 1
    let variable_arg_1 = a:{index} " same as a:1
    return variable_arg_1
endfunction
```

# 插件开发
## \<SID\>含义
[Vim 脚本学习笔记 · 幽谷奇峰 | 燕雀鸣幽谷，鸿鹄掠奇峰](https://yysfire.github.io/vim/vimscript-note.html)

`<SID>` 和 `<Plug>` 都是用来避免映射的键序列和那些仅仅用于其它映射的映射起冲突。  
`<Plug>` 在脚本外部是可见的。它被用来定义那些用户可能定义映射的映射，  
`<SID>` 是脚本的 ID，用来唯一的代表一个脚本。

## 命令补全
[wsdjeg/vim-plugin-dev-guide: Vim 插件开发指南](https://github.com/wsdjeg/vim-plugin-dev-guide)

`command! -nargs=* -complete=custom,helloworld#complete HelloWorld call helloworld#test()`  
其中 `-complete=custom,helloworld#complete` 表示，改命令的补全方式采用的是自定义函数 helloworld#complete。

## vim运行时目录
插件的目录，可参考 vim 本身安装的运行时目录。所谓运行时目录，顾名思义，就是在 vim 运行时如果要加载 *.vim 脚本，应该到哪里找文件。
有两个相关的环境变量，可用如下命令查看：

```vim
:echo $VIM
:echo $VIMRUNTIME
```
如果从源码安装 vim ，且自定义安装于家目录的话，它们的值大概如下：
```
$VIM = ~/share/vim
$VIMRUNTIME = ~/share/vim/vim81
```
所以 $VIM 指的是 vim 安装目录，而且不同版本的 vim 都将安装在该目录下，$VIM-RUNTIME 就是具体当前运行的 vim 版本的安装目录。不过此安装目录不包括 vim 程序本身（那是被安装到 ~/bin 中的），主要是 vim 运行时所需的大量 *.vim 脚本，相当于“官方插件”。该目录有哪些文件目录，可用如下命令显示：
```
:!ls -F $VIMRUNTIME
```
$VIMRUNTIME 既是官方目录，显然是不建议用户在其内修改或增删的。如果不是自定义安装在个人家目录，使用系统默认安装的 vim 的话，普通用户也无权修改。于是 vim 提供了一个选项叫 **&runtimepath （常简称 &rtp）**，那是类似系统 shell 的环境变量 $PATH，就是一组目录，只不过不用冒号分隔，而是用逗号分隔。可用如下命令查看 &rtp ：
```
:echo &rtp
:echo split(&rtp, ',')
```
通常，~/.vim/ 目录会在 &rtp 列表中，而且往往是第一个。另外，官方目录 $VIMRUN-TIME 也在 &rtp 列表较后一个位置。当 vim 在运行时需要加载脚本时，就会依次从 &rtp列表中每个目录（及其子目录）中查找，有时查找第一个就会停止。所以 $VIMRUNTIME目录并不特殊，只是 &rtp 中一个优先级并不高的目录。对用户来说，~/.vim/ 目录才更特殊些，常被称为 vim 的用户目录。

一般建议用户将个人的 vimrc 及其他 vim 脚本放在 ~/.vim/ 目录中。可以用这个命令：
```
:echo $MYVIMRC
```
查看当前你运行的 vim 启动时读取 vimrc。如果显示是 ~/.vimrc ，则建议将其移至 ~/.vim/vimrc 或软链接指向它。vim 会尝试读取 vimrc 的几个位置及顺序，也可用如下命令查看：
```
:version
```