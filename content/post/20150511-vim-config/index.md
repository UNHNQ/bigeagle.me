---
title: "我的 Vim 配置"
date: 2015-05-11 23:46:44
slug: vim-config
tags: 
  - linux 
  - vim
  - super user
---

我的 Vim 配置成型已经有不少时间了，过去也是看着 [依云](http://lilydjwg.is-programmer.com) 等人的配置综合而成的，
再加上有人黑我一年不写blog，一写果然就是 Arch 挂，因此决定多分享一些 [super user](/tags/super-user) 系列的内容。

我的 vim 所有配置文件，均放在 [GitHub](https://github.com/bigeagle/neovim-config) 上，但它很可能不适合别人，因此
这篇文档主要讲我的配置文件组织方式、各个插件的用途等等，方便读者组织自己的配置文件。

废话少说，先上王道。

{{<img src="vim-screenshot.png" class="center" >}}

<!--more-->

# 先期准备 & 前言废话

首先你需要一个 `vim`, 一般这个 `vim` 需要加上 `+python +ruby` 等支持才能支持更多插件，如果使用 Arch Linux 的话，建议安装 `gvim`。另外目前很多 vim
插件通过 github 分发，因此 git 也是强烈建议安装的。

本文中所说的 vim 就是 vim，不是 vi，也不是 [neovim](http://neovim.io/)，前者是 vim 的祖先，后者是号称「下一代vim」的一个正在开发中的 fork。
（事实上我正在用 neovim ，但目前还没有什么 killer feature, bug 也比较多，目前还没太大必要值得一般人用）

其次，我仍然十分建议在终端中使用 vim，毕竟很多操作，比如编译、测试，还是用 shell 舒服。配合利器 Tmux，在一个 tab 中编辑，另一个 tab 中编译/测试，这样是比较舒服的。
当然，你可以配合一些插件在 vim 中打开一个 shell，但总还是不太爽……


21 世纪一般同学都使用图形终端了，目前绝大多数图形终端都支持 256 色，甚至 24 位真彩色。目前 vim 在终端下最多还只支持 256 色，neovim 支持真彩色，但是我也只是用到 256。
如何判断自己的终端色彩数量呢？敲一个
```
$ tput colors
```
如果你没有特意配置过，返回的值应该是 `8`，也就是默认色彩数。我个人比较建议的方式是，在 `~/.bashrc` 或者 `~/.zshrc` 中加入
```bash
case "$TERM" in
    xterm)
        export TERM=xterm-256color
        ;;
    screen)
        export TERM=screen-256color
        ;;
esac
```
这样的话，当终端是 `xterm` 的时候，shell 会让终端打开 256 色支持。

# 插件系统、整体配置

vim 本身实现了一种脚本语言 VimL 用于 vim 配置、编写插件，VimL 我也不会，所以不讲了…… 需要说明的是，vim 本身的功能只是一个足够好的编辑器，
但实际应用中，人们常常还需要一些高级编辑，或编辑之外的功能，这些都需要通过插件实现。

配置文件呢，有多个默认位置，和用户相关的主要就是`~/.vimrc` 和 `~/.vim/`，前者是配置文件，后者是各类插件目录。在一切之前，请先确保文件/目录准备完毕

```
$ mkdir ~/.vim
$ touch ~/.vim/vimrc
$ ln -s ~/.vim/vimrc ~/.vimrc  # 把 vimrc 本体放在 ~/.vim/ 中，方便管理
```

过去的 vim 插件安装方式是一股脑全部装在 `~/.vim` 里，插件多了之后管理起来极其困难。好在后来有了 pathogen，以及衍生出来的 vundle 等等，
目前 vim 的插件管理已经非常轻松了。我使用 <s>[Vundle](https://github.com/gmarik/Vundle.vim)</s> [vim-plug](https://github.com/junegunn/vim-plug) 
作为插件管理器，比起 vundle 功能更全面，且支持并行下载。

## 安装 vim-plug 

由于 Vim 并不自带插件管理，因此我们要手动安装配置。vim-plug  的安装很简单，首先把 `plug.vim` 下载了放在 `~/.vim/autoload/` 下

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
  https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

就好了。

## 文件结构

我们的 `~/.vim` 目录整体会长成这样: 

```
+-- vimrc   # 主配置文件
|
+---config/  # 存放各类插件的配置文件
|   \-- xxx.vim
|   \-- ooo.vim
|   \-- ....
+--- bundle/ # 存放各类插件的代码，交给 Vundle 管理就好了
```

所以我们需要关心的也就是 `vimrc` 和 `config/` 里的各类配置。

## vimrc 结构

vimrc 大概会长成这样

```vim
call plug#begin('~/.vim/bundle') " vim-plug 初始化

"""
" 插件列表
"""

" 必须使用单引号
Plug 'rking/ag.vim'
Plug 'kien/ctrlp.vim'
Plug 'Yggdroot/indentLine'
Plug 'Valloric/MatchTagAlways'
Plug 'Valloric/YouCompleteMe'

" 还有好多插件

" 结束插件列表
call plug#end()


" Vim 基础配置部分

" 基础配置结束

" 插件配置部分

source ~/.vim/config/python-mode.vim
source ~/.vim/config/tagbar.vim
source ~/.vim/config/nerdtree.vim
source ~/.vim/config/syntastic.vim  
" ....
```

所以配置文件分为三个部分：
- 插件列表
- 基础配置
- 各插件配置

# 基础配置

我先把基础配置，带注释发上来，再贴插件列表

{{< gist bigeagle ab159ef3d3d0828436a0 >}}

## 有用的插件

以下所有插件名，例如 `rking/ag.vim` 改成 `Plugin 'rking/ag.vim'` 的形式放在 vimrc 的插件段，即可安装使用。粗体的为强烈推荐安装。

* rking/ag.vim 使用 SilverSearcher 搜索代码，比 grep 速度快，稳准狠
* **kien/ctrlp.vim** 使用模糊匹配搜索/打开文件，非常好用，强烈推荐
* **Yggdroot/indentLine** 编辑 python 等用空格缩进的文件时，显示缩进标记线
* **Valloric/YouCompleteMe** 极其强大的自动补全、C/C++/C#代码分析器
* **jlanzarotta/bufexplorer** 快速打开历史编辑过的文件
* mattn/emmet-vim 曾叫 ZenCoding，对 HTML/XML 编辑很有用
* bigeagle/molokai 一个很漂亮的颜色主题，我修改过，御用
* **scrooloose/nerdtree** 截图中左侧文件列表
* **scrooloose/syntastic** 语法检查器，非常有用
* klen/python-mode 对 python 程序员，强烈推荐
* **kien/rainbow_parentheses.vim** 使用不同颜色标记各级括号，非常好
* **majutsushi/tagbar** 截图中左侧的类/函数/方法列表
* SirVer/ultisnips 和 honza/vim-snippets : 一些 snippets，省时省力，只是常常记不住
* **bling/vim-airline** 截图中下方非常漂亮的状态栏
* fatih/vim-go Go 程序员必备
* lervag/vim-latex $\LaTeX$ 编辑必备
* **jrosiek/vim-mark** 阅读代码必备，给单词加 mark 高亮

写完插件列表之后，打开 vim 然后 `:PlugInstall` 即可完成插件下载、安装，有些插件（如 YouCompleteMe) 需要进一步安装的，请查看插件文档。

插件的配置，一般插件文档写得很详细，我就不多说了。


# 直接用我的配置文件

如果直接用我写好的配置呢，我建议在 github 上 fork，因为我的配置是面向 neovim 的，所以你需要把所有 `.nvim` 替换成 `.vim`，然后插件列表可能不适合你，最好也
根据自己的需求做一些调整。

以上。

# Revision

- 2015.06.22 抛弃 Vundle，转向 vim-plug

