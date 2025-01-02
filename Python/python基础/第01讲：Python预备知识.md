# Python 预备知识

## 代码注释

Python 中有两种形式的注释：

1. 单行注释：以 `#` 和空格开头，可以注释掉从 `#` 开始后面一整行的内容。
2. 多行注释：三个引号开头，三个引号结尾，通常用于添加多行说明性内容。(可以是双引号也可以是单引号)

```python
第一个注释

'''
第二注释
第三注释
'''
 
"""
第四注释
第五注释
"""
print ("Hello, Python!")
```

## 标识符

标识符命名规则：

- 第一个字符必须是字母表中字母或下划线 `_` 
- 标识符的其他的部分由 **字母**、**数字** 和 **下划线** 组成。
- 标识符对大小写敏感。

> [!NOTE]
>
> 在 Python 3 中，可以用中文作为变量名，非 ASCII 标识符也是允许的了。



## 关键字

查看 python 关键字命令：

```python
import keyword
keyword.kwlist
```

打印如下：

```python
['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```


## 行与缩进

python 使用缩进来表示代码块。缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数。缩进不一致，会导致运行错误。

```python
if True:
    print ("True")
else:
    print ("False")
```


## 多行语句

Python 通常是一行写完一条语句，但如果语句很长，我们可以使用反斜杠 `\` 来实现多行语句

```python
total = item_one + \
        item_two + \
        item_three
```

在 `[]`, `{}`, 或 `()` 包裹的多行语句中，不需要使用反斜杠 `\`，例如：

```python
total = ['item_one', 'item_two', 'item_three',
        'item_four', 'item_five']
```

> [!NOTE]
>
> 正斜杠 `/` 通常用于链接、文件路径等场景，表示目录分隔符。反斜杠 `\` 则主要用于编程语言中，作为转义字符，用于表示特殊字符。



## 文本输出

`print` 函数输出默认是换行的。如果要实现不换行，需要在变量末尾加上 `end=""`

```python
x="a"
y="b"
# 换行输出
print( x )
print( y )
 
print('-----------------')
# 不换行输出
print( x, end=" " )
print( y, end=" " )
print()
```

## 代码组

缩进相同的一组语句构成一个代码块，我们称之代码组。像 `if`、`while`、`def` 和 `class` 这样的复合语句，首行以关键字开始，以冒号 ( `:` ) 结束，该行之后的一行或多行代码构成代码组。

我们将首行及后面的代码组称为一个子句。如下实例:

```python
if expression : 
   suite
elif expression : 
   suite 
else : 
   suite
```


## 等待输入

执行下面的程序在按回车键后就会等待用户输入：

```python
input("\n\n按下 enter 键后退出。")
```

以上代码中，`\n\n` 在结果输出前会输出两个新的空行。一旦用户按下 enter 键时，程序将退出。


## 同一行执行多条语句

Python 可以在同一行中使用多条语句，语句之间使用分号 `;` 分割，示例如下：

```python
import sys; x = 'runoob'; sys.stdout.write(x + '\n')
```



## 模块导入

在 python 用 `import` 或者 `from... import` 来导入相应的模块。

- 将整个模块 (somemodule) 导入，格式为： `import somemodule`
- 从某个模块中导入某个函数, 格式为： `from somemodule import somefunction`
- 从某个模块中导入多个函数, 格式为： `from somemodule import firstfunc, secondfunc, thirdfunc`
- 将某个模块中的全部函数导入，格式为： `from somemodule import *`