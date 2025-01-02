提高自己vim最佳使用体验就必须要安装多个插件。这里主要使用的插件管理器是 vim-plug 

# 代码补全插件：coc.nvim

注意：确保 `Vim >= 8.1.1719` or `Neovim >= 0.4.0.` and `nodejs >= 16.18.0`

**1、前提是要安装 [clangd服务](https://github.com/clangd/clangd/releases/tag/17.0.3)  [nodejs服务](https://nodejs.org/en/download/)**

在 `.profile` 文件里设置环境变量

```shell
# clangd 17
export PATH=$PATH:~/Tools/clangd_17.0.3/bin
export PATH=$PATH:~/Tools/node/bin
```

**2、安装 coc.nvim 插件** 

打开 `.vimrc` 文件，在 `call plug` 的作用域里添加[安装语句](https://github.com/neoclide/coc.nvim)：

```
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

**3、配置支持的编程语言**

clangd服务与coc扩展一起使用才能支持 LSP，所以打开vim输入命令：

```shell
# 这是安装json的语言
:CocInstall coc-json coc-tsserver

# 安装C/C++
:CocInstall coc-clangd coc-tsserver
```

或者，可以在 coc-settings.json 中配置语言服务器（使用 :CocConfig 打开），[更多语言支持请点击](https://github.com/neoclide/coc.nvim/wiki/Language-servers)。

```
{
  "languageserver": {
    "go": {
      "command": "gopls",
      "rootPatterns": ["go.mod"],
      "trace.server": "verbose",
      "filetypes": ["go"]
    }
  }
}
```

**4、配置补充**

Coc.nvim取消变量名自动提示：

```json
{
    "inlayHint.enable": false
}
```


# 主题插件：gruvbox

```shell
Plug 'morhetz/gruvbox'

# 自动去查找主题
autocmd vimenter * nested colorscheme gruvbox
se t_Co=256
set bg=dark
```

