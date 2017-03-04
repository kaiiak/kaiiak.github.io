title: 自用vim配置文件
date: 2016-03-05 16:49:30
tags: [Linux,vim]
---
# 使用`Pathogen`管理插件
推荐[这篇教程](http://lostjs.com/2012/02/04/use-pathogen-and-git-to-manage-vimfiles/)，使用`Pahtogen`+`Git`的方式管理插件和vim配置，方便vim的迁移和团队配置的统一。

# 自用的vim配置文件
```
" 编码
set encoding=utf-8

" 避免以前版本的一些bug和局限
set nocompatible

" 逐步搜索模式，对当前键入的字符进行搜索而不必等待键入完成,/b寻找b开头的。
set incsearch

"自动识别文件类型
" set filetype

" 搜索时高亮显示找到文本
set hlsearch

" 在Insert模式下退格键何时可以删除光标之前的字符。三项内容分别指定了Vim可以删除位于行首的空格,断行,以及开始进入Insert模式之前的位置。
set backspace=indent,eol,start


" 设置tab键为4个空格，设置当行之间交错时使用4个空格
set tabstop=4
set shiftwidth=4

" 匹配模式，左括号匹配右括号
set showmatch

" 在覆盖一个文件之前备份该文件。但是对VMS系统除外,因为该系统已经为文件保存了老的版本。备份文件名由当前文件名加后辍"~"组成。
" if has("vms")
" set nobackup
" else
" set backup
" endif

" 设置冒号命令和搜索命令的命令历史列表的长度。
set history=1000

" 总是在Vim窗口的右下角显示当前光标的行列信息。
set ruler

" 在Vim窗口的右下角显示一个完整的命令已经完成的部分。比如说你键入"2f",Vim就会在你键入下一个要查找的字符之前显示已经键入的"2f"。一旦你接下来再键入一个字符比如"w",那么一个完整的命令"2fw"就会被Vim 执行,同时刚才显示的"2f"也将消失。
set showcmd

" 语法高亮
syntax on

" 显示行号
set nu

" 显示空格和TAB
set list
set listchars=tab:>-,trail:-

" 凸显当前行
set cursorline

" 检测文件类型
filetype on

" 启用鼠标
set mouse=a

" 配色方案
"colorscheme torte

" 以下为插件

" 管理插件的插件pathogen
call pathogen#infect()

" Powerline
set guifont=PowerlineSymbols\ for\ Powerline
set nocompatible
set laststatus=2
set t_Co=256
let g:Powerline_symbols = 'fancy'

" NERDTree
map <F10> :NERDTreeToggle<CR>

" C的编译和运行 
map <F5> :call CompileRunGcc()<CR> 
func! CompileRunGcc() 
exec "w" 
exec "!gcc % -o %<" 
exec "! ./%<" 
endfunc 

" C++的编译和运行 
map <F6> :call CompileRunGpp()<CR> 
func! CompileRunGpp() 
exec "w" 
exec "!g++ % -o %<" 
exec "! ./%<" 
endfunc 
```

# 插件列表

|插件名称|项目地址|教程地址|
|--------|:-------------|:-----------------|
|`pathogen`|[项目地址](http://www.vim.org/scripts/script.php?script_id=2332)|[教程地址](https://github.com/tpope/vim-pathogen)|
|`NERDTree`|[项目地址](http://www.vim.org/scripts/script.php?script_id=1658)|[教程地址](http://my.oschina.net/VASKS/blog/388907?fromerr=ktE2belY)|