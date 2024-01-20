---
title: "nvim配置教程"
date: 2023-08-16T16:05:25+08:00
tags: ["Vim"]
categories: []
draft: false
toc: true
---

# 1.安装NerdFont字体
[ryanoasis/nerd-fonts: Iconic font aggregator](https://github.com/ryanoasis/nerd-fonts)  
[How do I install fonts? - Ask Ubuntu](https://askubuntu.com/questions/3697/how-do-i-install-fonts)  
[bash - MobaXTerm Powerline Font Issue - Super User](https://superuser.com/questions/1134268/mobaxterm-powerline-font-issue/1251965)  

没有root权限时，ttf文件放在`~/.fonts`或者`~/.local/share/fonts`目录下  
有root权限时，放在`/usr/local/share/fonts`目录下
```shell
# 安装字体
fc-cache -fv
fc-list | grep <name-of-font>
```

# 2.掘金小册配置
## 2.1.目录结构
首先 init.lua 是整个配置的入口文件，负责引用所有其他的模块，基本上想要打开或关闭某个插件只要在这里修改一行代码即可。  

basic.lua： 基础配置，是对默认配置的一个重置。  
colorscheme.lua： 我们安装的主题皮肤配置，在这里切换主题。  
keybindings.lua： 快捷键的设置，所有插件的快捷键也都会放在这里。  
plugins.lua： 插件安装管理，插件安装或卸载全在这里设置。  
lsp 文件夹： 是对 Neovim 内置 LSP 功能的配置，包括常见编程语言与语法提示等。  
config ： 文件夹包含各种语言服务器单独的配置文件。  
setup.lua ： 内置 LSP 的配置。  
cmp.lua ： 语法自动补全补全的配置，包括各种补全源，与自定义代码段。  
ui.lua： 对内置 LSP 功能增强和 UI 美化。  
formatter.lua： 独立代码格式化功能。  
plugin-config 文件夹： 是对第三方插件的配置，未来每添加一个插件，这里就多一个配置文件。  
utils 文件夹： 是对常见问题的修改，包括输入法切换，针对 windows 的特殊配置等。  

## 2.2.手动下载插件管理器
```shell
git clone --depth 1 https://github.com/wbthomason/packer.nvim\
 ~/.local/share/nvim/site/pack/packer/start/packer.nvim
```

`PackerSync`下载插件

Neovim 推荐将数据存储在 标准数据目录下（:h base-directories 查看详细文档），标准数据目录默认是 ~/.local/share/nvim/ ，你可以通过调用 :echo stdpath("data") 命令查看你系统下的实际路径。
Packer 会将插件默认安装在 标准数据目录/site/pack/packer/start 中，完整目录也就是~/.local/share/nvim/site/pack/packer/start 目录下。

## 2.3.安装插件
### 侧边栏

### 状态栏

### 标签页

### 模糊搜索
telescope 依赖 ripgrep 和 fd-find
```shell
#riggrep
curl -LO https://github.com/BurntSushi/ripgrep/releases/download/12.1.1/ripgrep_12.1.1_amd64.deb
sudo dpkg -i ripgrep_12.1.1_amd64.deb

#fd-find
wget https://ghproxy.com/https://github.com/sharkdp/fd/releases/download/v8.5.3/fd_8.5.3_amd64.deb
sudo dpkg -i fd_8.5.3_amd64.deb
```

### 语法高亮
Tree-sitter 是一个解析器生成器工具和增量解析库，它可以在源文件编辑的同时高效的实时生成语法树。

`:TSInstallInfo` 命令查看 language parsers 列表与安装状态。  
`:TSInstall <language_to_install>` 调用 TSInstall 命令的时候，插件会在目录`~/.local/share/nvim/site/pack/packer/start/nvim-treesitter/parser`生成一个 <language>.so 语法文件。  
`:TSUninstall <language_to_uninstall>` 命令用于卸载 language parser 。  
`:TSModuleInfo` 命令来查看你的模块是否开启成功。  
`:TSBufToggle highlight` 命令可以切换打开关闭代码高亮功能。  