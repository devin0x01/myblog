---
title: "vim配置教程"
date: 2023-08-25T16:05:25+08:00
tags: ["Vim"]
categories: []
draft: false
toc: true
---

# 编译vim9
[vim/vim: The official Vim repository](https://github.com/vim/vim)
```
./configure --with-luajit --enable-pythoninterp=yes --enable-python3interp=yes \
    --enable-multibyte --prefix=$HOME/.local/vim9
make -j
make install
```

# 插件
## 1.vim-plug
插件管理
[junegunn/vim-plug: :hibiscus: Minimalist Vim Plugin Manager](https://github.com/junegunn/vim-plug)
```shell
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

- PlugInstall [name ...]
- PlugUpdate [name ...]
- PlugClean[!]
- PlugUpgrade: Upgrade vim-plug itself  
- PlugStatus: Check the status of plugins  
- PlugDiff: Examine changes from the previous update and the pending changes  
- PlugSnapshot[!] [output path]: Generate script for restoring the current snapshot of the plugins  

[vim-plug 安装失败_vimplus安装失败_云梦谭的博客-CSDN博客](https://blog.csdn.net/yetyongjin/article/details/121612373)
```shell
$ diff plug.vim plug.vim.bak
778c778
<       let fmt = get(g:, 'plug_url_format', 'https://git::@ghproxy.com/https://github.com/%s.git')
---
>       let fmt = get(g:, 'plug_url_format', 'https://git::@github.com/%s.git')
1174c1174
<             \ '^https://git::@ghproxy.com\.com', 'https://ghproxy.com/https://github.com', '')
---
>             \ '^https://git::@github\.com', 'https://github.com', '')
```

## 2.NERDTree
文件浏览器
[preservim/nerdtree: A tree explorer plugin for vim.](https://github.com/preservim/nerdtree)
```vim
"官方推荐的map
nnoremap <leader>n :NERDTreeFocus<CR>
nnoremap <C-n> :NERDTree<CR>
nnoremap <C-t> :NERDTreeToggle<CR>
nnoremap <C-f> :NERDTreeFind<CR>
```

| 命令 | 描述 |
| :-    | :- |
| o     | 打开文件并将焦点移动到打开的文件或展开当前文件夹 |
| Enter | 跟o一样 |
| go    | 跟o一样，但将焦点留在NerdTree |
| t     | 在新tab中打开文件 |
| T     | 同t，但保留焦点 |
| i     | 在一个新的 split window中打开文件 |
| gi    | 同i，保留焦点 |
| s     | 在新的 vsplit 窗口打开文件 |
| gs    | 同s保留焦点 |
| O     | 递归打开当前文件夹 |
| x     | 关闭当前文件夹的父文件夹 |
| X     | 递归关闭当前文件夹 |
| P     | 跳到根目录 |
| p     | 跳到当前目录的父目录 |
| q     | 退出NerdTree |

## 3.vim-devicons
首先需要安装Nerd Fonts，比如可以安装[ryanoasis/nerd-fonts](https://github.com/ryanoasis/nerd-fonts/releases/tag/v3.1.1)里面的FiraCode

这个插件用于支持Nerd Fonts
[ryanoasis/vim-devicons: Adds file type icons to Vim plugins such as: NERDTree, vim-airline, CtrlP, unite, Denite, lightline, vim-startify and many more](https://github.com/ryanoasis/vim-devicons)

```vim
set encoding=UTF-8

call plug#begin()
Plug 'ryanoasis/vim-devicons'
call plug#end()
```

## 4.ctags
[vim+ctags+cscope+Taglist+Nerdtree打造成sourceinsight - 知乎](https://zhuanlan.zhihu.com/p/85040099)  
ctags文件只能查看函数、类或变量的定义，而没有被调用信息。如果要知道一个函数在什么地方被使用，需要使用cscope工具。  
添加的tags最好是source code的索引，对于头文件索引没有效果。  

```shell
sudo apt install universal-ctags
# 生成ctag文件
ctags -R --c++-kinds=+p --fields=+iaS --extra=+q
```
ctags的选项:
>-R 表示递归创建，也就包括源代码根目录（当前目录）下的所有子目录;  
--c++-kinds=+p 是为c/c+语言添加函数原型信息;  
--fields=+iaS 是为标签添加继承信息（inheritance），访问控制信息（access）和函数特征（Signature）如参数表或原型等;  
--extra=+q 是为类成员添加标签;  

当用户在当前目录中运行vi时，会自动载入此tags文件。假如你想让你当前目录文件中的函数名在**其他目录**中打开vim时也能被定位到的话，那么可以把当前目录的tags文件路径添加到.vimrc中: `set tags+=/home/ubuntu/code/tags`

Ctrl＋] 跳到当前光标下单词的标签  
Ctrl＋O 返回上一个标签  
Ctrl＋T 返回上一个标签  
:tag TagName 跳到TagName标签  
以上命令是在当前窗口显示标签，当前窗口的文件替代为包标签的文件，当前窗口光标跳到标签位置。  

:stag TagName 新窗口显示TagName标签，光标跳到标签处  
Ctrl＋W + ] 新窗口显示当前光标下单词的标签，光标跳到标签处  
:tselect TagName  
输入以上命令后，vim会为你展示一个选择列表。然后你可以输入要跳转到的匹配代号 (在第一列)。其它列的信息可以让你知道标签在何处被定义过。  

:ptag TagName 预览窗口显示TagName标签，光标跳到标签处  
Ctrl＋W + } 预览窗口显示当前光标下单词的标签，光标跳到标签处  
:pclose 关闭预览窗口  
以上命令将在预览窗口显示标签。  

## 5.cscope
```shell
sudo apt install cscope
cscope -Rbkq
```
cscope的选项:
>-R: 在生成索引文件时，搜索子目录树中的代码  
-b: 只生成索引文件，不进入cscope的界面  
-q: 生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度  
-k: 在生成索引文件时，不搜索/usr/include目录  
-i: 如果保存文件列表的文件名不是cscope.files时，需要加此选项告诉cscope到哪儿去找源文件列表。可以使用“-”，表示由标准输入获得文件列表。  
-I dir: 在-I选项指出的目录中查找头文件  
-u: 扫描所有文件，重新生成交叉索引文件  
-C: 在搜索时忽略大小写  
-P path: 在以相对路径表示的文件前加上的path，这样，你不用切换到你数据库文件所在的目录也可以使用它了。  

生成.out文件之后，我们需要在当前用户的用户目录中的.vimrc文件中把.out数据库的路径配置进去，假如不配置的话，cscope无法查找.out所在目录文件中的函数等，使用cs add命令添加.out的路径，即在~/.vimrc文件中添加下面这些内容即可: `cs add /home/ubuntu/linux-4.12.1/linux/cscope.out`

```
USAGE   :cs find {querytype} {name}

    {querytype} corresponds to the actual cscope line
    interface numbers as well as default nvi commands:

        0 or s: Find this C symbol
        1 or g: Find this definition
        2 or d: Find functions called by this function
        3 or c: Find functions calling this function
        4 or t: Find this text string
        6 or e: Find this egrep pattern
        7 or f: Find this file
        8 or i: Find files #including this file
        9 or a: Find places where this symbol is assigned a value

For all types, except 4 and 6, leading white space for {name} is
removed.  For 4 and 6 there is exactly one space between {querytype}
and {name}.  Further white space is included in {name}.

EXAMPLES
    :cscope find c vim_free
    :cscope find 3 vim_free
```

## 6.taglist
[vim-scripts/taglist.vim: Source code browser (supports C/C++, java, perl, python, tcl, sql, php, etc)](https://github.com/vim-scripts/taglist.vim)  
Taglist其实是一个vim的插件，能将当前vim打开的文件中函数名、变量名等在一个窗口中列出来，并支持通过列出的函数名实现跳转。
```vim
map <F2> :Tlist <CR>

let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
let Tlist_Use_Right_Window=1
```

进入vim后用命令:Tlist打开/关闭taglist窗口，或者使用:TlistToggle在打开和关闭间切换。  
可以用:TlistOpen打开taglist窗口，用:TlistClose关闭taglist窗口。  

## 7.winmanager
[anscoral/winmanager.vim: winmanager : A windows style IDE for Vim](https://github.com/anscoral/winmanager.vim)  
```vim
let g:winManagerWindowLayout='FileExplorer|TagList'
nmap wm :WMToggle<cr>
```
| 命令 | 描述 |
| :-    | :- |
| :WMToggle | 打开/关闭WinManager |
| \-        | 返回上一层目录 |
| c         | 使浏览目录成为vim当前工作目录 |
| d         | 创建目录 |
| D         | 删除当前光标下的目录或文件 |
| i         | 切换显示方式 |
| R         | 文件或目录重命名 |
| s         | 选择排序方式 |
| r         | 反向排序列表 |
| x         | 定制浏览方式, 使用你指定的程序打开该文件 |
| :help winmanager | 帮助信息 |

# 7.LeaderF
[Yggdroot/LeaderF: An efficient fuzzy finder that helps to locate files, buffers, mrus, gtags, etc. on the fly for both vim and neovim.](https://github.com/Yggdroot/LeaderF)  
[vim plugin介绍之LeaderF | Mingjian's Blog](https://retzzz.github.io/dc9af5aa/)  
```vim
"let g:Lf_WindowPosition = 'popup'
nnoremap <leader>f :Leaderf file<CR>
nnoremap <leader>rg :Leaderf rg<CR>
```

1.下载`ctags`并生成索引  
[universal-ctags/ctags: A maintained ctags implementation](https://github.com/universal-ctags/ctags)  
`:LeaderfTag`命令可以全局查找tag，其依赖ctags生成的tags文件。

2.下载`rg`并加入PATH  
[Releases · BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep/releases)  
下载文件名中包含musl的版本(`ripgrep-14.1.0-x86_64-unknown-linux-musl.tar.gz`)，这个是静态链接的，可以解决动态库版本不匹配的问题。

3.下载`gtags`并加入PATH  
[gtags下载](https://ftp.gnu.org/pub/gnu/global/)  
leaderf生成的gtags文件是在`~/.cache/LeaderF/gtags/`目录下。
```
./configure
make -j
make install
```
# 8.gutentags
[Vim 8 中 C/C++ 符号索引：GTags 篇 - 知乎](https://zhuanlan.zhihu.com/p/36279445)  
传统 ctags 系统虽和 vim 结合紧密，但只能查定义无法查引用，cscope 能查引用，但只支持 C 语言，C++都不支持，况且常年不更新。ctags 由于使用文本格式存储数据，虽用了二分查找，但打开 Linux Kernel 这样的大项目时，查询会有卡顿的感觉。

GTags （或者叫做 GNU GLOBAL）比起 ctags 来说，有几个主要的优点：
- 不但能查定义，还能查引用
- 原生支持 6 种语言（C，C++，Java，PHP4，Yacc，汇编）
- 扩展支持 50+ 种语言（包括 go/rust/scala 等，基本覆盖所有主流语言）
- 使用性能更好的本地数据库存储符号，而不是 ctags 那种普通文本文件
- 支持增量更新，每次只索引改变过的文件
- 多种输出格式，能更好的同编辑器相集成

gutentags 不但能根据文件改动自动生成 ctags 数据，还能帮我们自动更新 gtags 数据，还可以同时支持 ctags/gtags。  
[Releases · universal-ctags/ctags-nightly-build](https://github.com/universal-ctags/ctags-nightly-build/releases)  
```vim
" 如果使用 universal ctags 需要增加下面一行，老的 Exuberant-ctags 不能加下一行
let g:gutentags_ctags_extra_args += ['--output-format=e-ctags']
```
