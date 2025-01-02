# 变量简介

CMake 中的变量分为两种，一是 CMake 提供，二是自定义。CMake 变量的命名区分大小写，且在存储时都是字符串。

CMake 获取变量：`${变量名}` 

变量的基础操作是 set () 与 unset ()，但也可用 list 或 string

打印信息：

```cmake
message("hello")
```

获取 cmake 定义的变量值

```cmake
${CMAKE_VERSION}
```


# Set 命令

set 命令可以设置普通变量、缓存条目、环境变量三种变量的值，分别对应以下三种命令格式。

set 的值 `<value>...` 表示可以给变量设置 0 个或者多个值。当设置变量多个值时（大于 2 个），多个值会通过**分号**连接符连接成一个真实的值赋值给变量；当设置 0 个值时，实际上是把变量变为未设置状态，相当于调用 unset 命令。

```cmake
set(<variable> <value>... [PARENT_SCOPE]) #设置普通变量
 
set(<variable> <value>... CACHE <type> <docstring> [FORCE]) #设置缓存条目
 
set(ENV{<variable>} [<value>]) #设置环境变量
```

- 如果在函数中使用 set，则 set 的范围只在该函数内，调用它的函数并不能获取到该变量，类似于 C++变量的作用域。
- 如果是在调用函数中定义变量，子函数还是可以获取到该变量。

更多 set 的使用细节请查看：[cmake 命令之set](https://blog.csdn.net/sinat_31608641/article/details/123101969) 





# List 方法命令

![[2_CMake 语法.png|600]]

```cmake
cmake_minimum_required(VERSION 3.20)


set(V_SET "s1" "s2" "s3")

list(APPEND V_LIST "l1" "l2" "l3")

# 获取长度
list(LENGTH V_SET set_len)
list(LENGTH V_LIST list_len)

message("set 的元素：${V_SET} 长度：${set_len}")
message("list 的元素：${V_LIST} 长度：${list_len}")
```

打印结果如下：

```shell
set 的元素：s1;s2;s3 长度：3
list 的元素：l1;l2;l3 长度：3
```


# 流程控制

if 条件流程控制；loop 循环流程控制：break、continue

## if 使用

```cmake
if(<condition>)
    ...
else()
    ...
endif()
```

与或非：NOT、OR、AND

大小余判断：LESS、EQUAL

## for 循环使用

定义：

```cmake
foreach(<loop_var> RANGE <max>)
    <commands>
endforeach()

foreach(<loop_var> RANGE <min <max> [<step>])
foreach(<loop_variable> IN [LISTS <lists>] [ITEMS <items>])
```

使用：

```cmake
# 第一种使用
foreach(VAR RANGE 3)
    message(${VAR})
endforeach()
# 打印结果是0、1、2、3
```

```cmake
# 第二种使用
set(MY_LIST 1 2 3)
# LISTS 固定写法
# ITEMS 4 f：追加遍历4与f
foreach(VAR IN LISTS MY_LIST ITEMS 4 f)
    message(${VAR})
endforeach()
```

```cmake
# 第三种使用：
set(L1 one two three four)
set(L2 1 2 3 4 5)
# ZIP_LIST 固定用法
foreach(num IN ZIP_LIST L1 L2)
    message("word = ${num_0}, num = ${num_1}")
endforeach()

# 打印结果：
# word = one, num = 1
# word = two, num = 2
# word = three, num = 3
# ...
```



# 函数

定义函数的语法：

```cmake
function(<name> [<argument>...])
    <commands>
endfunction()
```

使用：

```cmake
# 
function(MyFunc FirstArg)
    message("Function Name: ${CMAKE_CURRENT_FUNCTION}")
    message("FirstArg: ${FirstArg}")
    set(FirstArg "New value")
    message("FirstArg again: ${FirstArg}")
    message("ARGV0: ${ARGV0}")
    message("ARGV1: ${ARGV1}")
    message("ARGV2: ${ARGV2}")
endfunction()

set(var "First value")
MyFunc(${var} "Second value")
message("FirstArg: ${var}")
```

打印结果：

```shell
Function Name: MyFunc
FirstArg: First value
FirstArg again: New value
ARGV0: First value
ARGV1: Second value
ARGV2:
FirstArg: First value
```

结论：函数可以接收多个参数，且以存储在 `ARGV[nums]` 中。函数内部是无法改变外部变量的值，只能是覆盖。


# Scope 作用域

cmake 有两种作用域：

1. Function Scope 函数作用域
2. Directory Scope 文件夹作用域：当从 add_subdirectory () 命令执行嵌套目录中的 CMakeLists. txt 列表文件时，注意父 CMakeLists. txt 其中的变量是可以传递给子 CMakeLists. txt 使用的



# 宏

cmake 中的宏

```cmake
macro(<name>[<argument>...])
	<commands>
endmacro()
```

注意：尽量不要写宏，只要会读就好。

使用：

```cmake
macro(Test myVar)
	# 创建了一个新的myVar变量
	set(myVar "New value")
	message("argument: ${myVar}")
endmacro()

set(myVar "First value")
message("myVar: ${myVar}")
Test("Second value")
message("myVar: ${myVar}")
```

执行结果如下：

```shell
myVar: First value
argument: Second value
myVar: New value
```

1、一开始在第 7 行设置变量并赋值，所以正常打印第一条结果是正确的。

2、调用 Test 宏函数，将 Test 宏内的所有 myVar 的值替换为传递给宏的值（可以认为是 c++里的预编译），接着调用执行在第 3 行时，New value 会将 myVar 的值进行覆盖。

宏不像函数那样有着自己的作用域，所以是可以直接作用于外部的变量。
