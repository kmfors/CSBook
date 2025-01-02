# vim配置


```vim
" author: kmfors

if has("syntax")
  syntax enable               " 语法高亮，不会强制覆盖已存在的语法文件(语法插件)
  syntax on                   " 语法高亮, 会强制覆盖，由vim提供
endif

set nocompatible            " 使用vim默认配置，进入非兼容模式

set ic                      " 查询匹配下，忽略英文大小写

set wrap                    " 超出窗口边界处自动换行显示文本

set ruler                   " 右下角显示当前光标位置

set number                  " 显示行号

set cindent                 " C 语言风格的代码块缩进

set showmatch               " 高亮括号配对

set expandtab               " 告诉vim使用tab时，是要插入空格字符而不是制表符字符
set tabstop=4               " 使用tab时，每个制表符所占据的列数

" 键盘Tab缩进时，每个缩进所等价的空格数,按下后是插入4个空格而不是制表符
set softtabstop=4
set shiftwidth=4            " 设置自动缩进为 4 个空格


"set autoindent             " 自动缩进:与上一行保持一致的自动空格
set smartindent             " 加强版自动缩进

set scrolloff=6             " 设置光标离窗口上下 6 行时窗口自动滚动

set encoding=utf-8          " utf-8 编码

" 退格删除光标前面的字符
set backspace=indent,eol,start

set wildmenu                " 状态行上显示补全匹配

" 按 Esc 的生效更快速
set ttimeout
set ttimeoutlen=100

autocmd BufReadPost * 
\ if line("'\"") > 0 | if line("'\"") <= line("$") |
\ exe("norm '\"") | else | exe "norm $"|endif|endif

set laststatus=2            " 设置状态栏在倒数第2行
" 设置状态栏格式
set statusline=%1*\%f                       "显示文件名
set statusline+=%=%2*\%m%r%h%w\ %*          "显示文件类型及文件状态
set statusline+=%3*\[%{&ff}]\[%{&fenc}]%y\ %*   "显示文件编码类型
set statusline+=%4*\%3p%%\%*                "显示光标前文本所占总文本的比例
set statusline+=%5*\%5l:%c\ %*              "显示光标所在行和列
hi User1 cterm=none ctermfg=25 ctermbg=0
hi User2 cterm=none ctermfg=208 ctermbg=0
hi User3 cterm=none ctermfg=169 ctermbg=0
hi User4 cterm=none ctermfg=100 ctermbg=0
hi User5 cterm=none ctermfg=green ctermbg=0


```

