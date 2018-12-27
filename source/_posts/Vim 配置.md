---
title: Vim 配置
date: 2017-12-02T03:54:48.000Z
tags: ['Linux']
categories: [Ops]
---
> 全局配置 **/usr/share/vim/vimrc**，不建议修改。建议修改当前用户配置 **~/.vimrc**，加载配置时会覆盖全局配置。

- 我的Vim配置，我只是轻量级用户，不像某些大佬（变态）

```bash
syntax on
set smartindent
set autoindent
set tabstop=4
set shiftwidth=4
set expandtab
set softtabstop=4
set ic "搜索忽略大小写
set hlsearch
set incsearch
set number
set ruler
set cursorline
set nocompatible "关闭vi兼容
set nowrap "关闭超屏自动换行
set encoding=utf-8
set fileencodings=utf-8,ucs-bom,GB2312,big5
filetype plugin indent on

nnoremap . $
nnoremap , ^
nnoremap <F2> :set number!<CR>
nnoremap <F3> :exec exists('syntax_on') ? 'syn off' : 'syn on'<CR>
nnoremap <F4> :set wrap!<CR>
nnoremap <F5> :set hlsearch!<CR>

"自动补全
:inoremap ( ()<ESC>i
        :inoremap ) <c-r>=ClosePair(')')<CR>
:inoremap { {<CR>}<ESC>O
    :inoremap } <c-r>=ClosePair('}')<CR>
    :inoremap [ []<ESC>i
    :inoremap ] <c-r>=ClosePair(']')<CR>
    :inoremap " ""<ESC>i
    :inoremap ' ''<ESC>i
function! ClosePair(char)
    if getline('.')[col('.') - 1] == a:char
    return "\<Right>"
    else
    return a:char
    endif
    endfunction
```

#### Tips

- 主要设置语法高亮、智能缩进、高亮搜索、显示行号、编码格式
- 配置快捷键映射，`,`跳转到行首`.`跳转到行尾，`F2`开关显示行号，`F3`开关语法高亮，`F4`开关超屏换行，`F5`开关高亮搜索
- `nnoremap`中n表示**normal**模式，i表示**insert**模式，nore表示**no recursion**非递归映射，`<CR>`表示回车
- 最后是设置智能补全
- **.rc(run commands)** 文件，通常在程序的启动阶段被调用

#### References

- [到底 VIM 能配置到多强大的程度？](https://www.zhihu.com/question/20151659)
[利器系列之 —— 编辑利器 Vim 之基础配置](http://blog.guorongfei.com/2015/08/31/vim-basic/)，非常好的博客，推荐关注