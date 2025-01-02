
- ctrl + f: 向下翻页
- ctrl + b: 向上翻页
- zt: 将光标所在行移动到窗口顶端
- zb: 将光标所在行移动到窗口底端
- zz: 将光标所在行移动到窗口中央
撤销在普通模式下使用 `u`，反撤销使用 `Ctrl+r`

本文到此就结束了，最后再来总结一下该文中使用到的命令和快捷键：

- bnext: 切换到缓冲区列表中的下一个缓冲区
- bprev: 切换到缓冲区列表中的下一个缓冲区
- blast: 切换到缓冲区列表中的最后一个缓冲区
- bfirst: 切换到缓冲区列表中的第一个缓冲区
- <Ctrl+^>: 在上一个被激活的缓冲区和当前被激活的缓冲区之间进行轮换
- args: 显示当前缓冲区列表参数。后面也可以跟上文件名、shell命令和通配符，设置对应的缓冲区列表参数
- next: 切换到缓冲区列表参数中的下一个缓冲区
- prev: 切换到缓冲区列表参数中的上一个缓冲区
- last: 切换到缓冲区列表参数中的最后一个缓冲区
- first: 切换到缓冲区列表参数中的第一个缓冲区

---


CTRL-^：只在一个缓冲区内切换不同文件。所有打开过的文件名都会保存在缓冲区列表中，供其切换

编辑文件（edit file）with `:e filename`    写入文件 with `:w filename`

CTRL-G `:f` `:file`  打印当前文件信息

`:ls` List all the currently known file names.

`:cd` display the full path name of a file

`:e <cfile>` 编辑新文件并打开

`:w <cfile>` 写入新文件不打开

`:map gf :e <cfile><CR>` gf映射（等价于）编辑新文件打开的命令

cursor keys光标键映射其他键

```vim
:cnoremap <C-A> <Home>
:cnoremap <C-F> <Right>
:cnoremap <C-B> <Left>
:cnoremap <Esc>b <S-Left>
:cnoremap <Esc>f <S-Right>
```

# nvim using lua

1、打印

```vim
:lua print("Hello!")
```

2、local设置foo变量赋值为1
```vim
:lua local foo = 1
:lua print(foo)
" prints "nil" instead of "1" error!变量也是有生命域的
```



```shell
~/.config/nvim
|-- after/
|-- ftplugin/
|-- lua/
|   |-- myluamodule.lua
|   |-- other_modules/
|       |-- anothermodule.lua
|       |-- init.lua
|-- plugin/
|-- syntax/
|-- init.vim
```

可以使用 :source 命令来运行外部文件中的 Lua 脚本

可以直接要求使用包含 init.lua 文件的文件夹，而无需指定文件名。
无需指定文件名：

require命令会自动寻找~/.config/nvim/lua下的lua配置文件
```lua
require("myluamodule")
```

加载目录下的lua配置文件
```lua
require('other_modules/anothermodule') -- 适用win
-- or
require('other_modules.anothermodule')
```

可以直接指定某个目录，自动导入当前的init文件
```lua
require('other_modules') -- loads other_modules/init.lua
```

要求使用不存在的模块或包含语法错误的模块会中止当前执行的脚本。
pcall() 可用于捕捉此类错误。下面的示例下面的示例尝试加载 module_with_error，如果成功，则调用其中一个函数，否则打印错误信息。函数，否则将打印错误信息：
```lua
local ok, mymod = pcall(require, 'module_with_error')
if not ok then
  print("Module had an error")
else
  mymod.function()
end
```

与 `:source` 不同，`require( )` 不仅会搜索 `runtimepath`下的所有 `lua/` 目录，还会在首次使用时进行缓存模块。因此，第二次调用 require() 时不会再次执行脚本，而是返回缓存文件。
要重新运行文件，需要先手动将其从文件缓存：
```lua
package.loaded['myluamodule'] = nil
require('myluamodule')    -- read and execute the module again from disk
```

### Vim commands

要通过 Lua 运行任意 Vim 命令，可将其作为字符串传递给 vim.cmd()：
```lua
vim.cmd("colorscheme habamax")
```

另一种方法是使用字面字符串（参见 lua-literal），以双括号 `[[ ]]` 分隔的字面字符串，如
```lua
vim.cmd([[colorscheme habamax]])
```

使用字面字符串的另一个好处是，它们可以是多行字符串；这样就可以在 vim.cmd() 的单次调用中传递多个命令：
```lua
vim.cmd([[
  highlight Error guibg=red
  highlight link Warning Error
]])
```

如果你想以编程方式创建 Vim 命令，下面的表格会很有用（所有这些都等同于上面的相应行很有用（所有这些都等同于上面的相应行）：
```lua
vim.cmd.colorscheme("habamax")
vim.cmd.highlight({ "Error", "guibg=red" })
vim.cmd.highlight({ "link", "Warning", "Error" })
```

### Vimscript functions

使用 vim.fn 从 Lua 调用 Vimscript 函数。Lua 和 Vimscript 之间的数据类型会自动转换：
```lua
print(vim.fn.printf('Hello from %s', 'Lua'))
local reversed_list = vim.fn.reverse({ 'a', 'b', 'c' })
vim.print(reversed_list) -- { "c", "b", "a" }
local function print_stdout(chan_id, data, name)
  print(data[1])
end
vim.fn.jobstart('ls', { on_stdout = print_stdout })
print(vim.fn.printf('Hello from %s', 'Lua'))
```

请注意，在 Lua 中，散列 (#) 并不是有效的标识符字符，因此例如，自动加载函数必须使用这种语法调用
```lua
vim.fn['my#autoload#function']()
```

## Variables

可以使用以下封装器设置和读取变量，这些封装器直接对应于变量范围直接与变量范围相对应：

vim.g：全局变量 (g:)
vim.b：当前缓冲区的变量 (b:)
vim.w：当前窗口的变量 (w:)
vim.t：当前标签页的变量 (t:)
vim.v：预定义的 Vim 变量 (v:)
vim.env：编辑器会话中定义的环境变量

数据类型会自动转换。例如
```lua
vim.g.some_global_variable = {
  key1 = "value",
  key2 = 300
}
vim.print(vim.g.some_global_variable)
--> { key1 = "value", key2 = 300 }
```

## Options

通过 Lua 设置选项有两种互补方式。

设置全局和本地选项（例如在 init.lua 中）最方便的方法是通过 vim.opt 和 friends：

[vim.opt](https://neovim.io/doc/user/lua.html#vim.opt): behaves like [:set](https://neovim.io/doc/user/options.html#%3Aset)

[vim.opt_global](https://neovim.io/doc/user/lua.html#vim.opt_global): behaves like [:setglobal](https://neovim.io/doc/user/options.html#%3Asetglobal)

[vim.opt_local](https://neovim.io/doc/user/lua.html#vim.opt_local): behaves like [:setlocal](https://neovim.io/doc/user/options.html#%3Asetlocal)

例如，Vimscript 命令
```vim
set smarttab
set nosmarttab
```

相当于
```lua
vim.opt.smarttab = true
vim.opt.smarttab = false
```

特别是，它们允许通过 Lua 表格轻松处理列表、地图和设置类选项：
```vim
set wildignore=*.o,*.a,__pycache__
set listchars=space:_,tab:>~
set formatoptions=njt
```

代替的是
```lua
vim.opt.wildignore = { '*.o', '*.a', '__pycache__' }
vim.opt.listchars = { space = '_', tab = '>~' }
vim.opt.formatoptions = { n = true, j = true, t = true }
```

vim.o：行为类似于 :set
vim.go：行为类似于 :setglobal
vim.bo：用于缓冲区范围的选项
vim.wo：用于窗口作用域选项（可双索引）

## Mappings

可以使用 vim.keymap.set() 创建映射。该函数需要三个参数：
- {mode} 是一个字符串或字符串表，包含映射生效的模式前缀。前缀是 :map-modes 中列出的前缀，或 :map! 中的"!"，或 :map 中的空字符串。
- {lhs} 是一个字符串，包含应触发映射的键序列。
- {rhs} 是包含 Vim 命令或 Lua 函数的字符串，当输入 {lhs} 时应执行该命令或函数。空字符串等同于 `<Nop>`，表示禁用按键。
```lua
-- Normal mode mapping for Vim command
vim.keymap.set('n', '<Leader>ex1', '<cmd>echo "Example 1"<cr>')
-- Normal and Command-line mode mapping for Vim command
vim.keymap.set({'n', 'c'}, '<Leader>ex2', '<cmd>echo "Example 2"<cr>')
-- Normal mode mapping for Lua function
vim.keymap.set('n', '<Leader>ex3', vim.treesitter.start)
-- Normal mode mapping for Lua function with arguments
vim.keymap.set('n', '<Leader>ex4', function() print('Example 4') end)
```

第四个可选参数是一个表，其中包含修改映射行为的键，如 :map-arguments 中的键。
例如 :map-arguments 中的键。以下是最有用的最有用的选项：

- buffer：如果给定，则只为指定编号的缓冲区设置映射；0 或 true 表示当前缓冲区。
```lua
-- set mapping for the current buffer
vim.keymap.set('n', '<Leader>pl1', require('plugin').action, { buffer = true })
-- set mapping for the buffer number 4
vim.keymap.set('n', '<Leader>pl1', require('plugin').action, { buffer = 4 })
```

- silent：如果设置为 "true"，则会抑制错误信息等输出。
```lua
vim.keymap.set('n', '<Leader>pl1', require('plugin').action, { silent = true })
```

- 