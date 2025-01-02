
```cmake
target_compile_options(<target> [BEFORE]
    <INTERFACE|PUBLIC|PRIVATE> [items1...]
    [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

这些选项将在编译给定的 `<target>` 时使用，而 `<target>` 必须是通过 add_executable() 或 add_library() 等命令创建的，注意：这些选项在链接目标时不会使用。
# target_compile_definitions

```cmake
target_compile_definitions(<target>
    <INTERFACE|PUBLIC|PRIVATE> [items1...]
    [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```    

编译目标文件 `<target>` 时要使用的编译定义（宏定义）。

被命名的 `<target>` 必须是通过 `add_executable()` 或 `add_library()` 命令创建的。

以下条目中的任何前导 `-D` 都将被删除。空条目是被忽略的。例如，下面的内容都是等价的：

```cmake
target_compile_definitions(foo PUBLIC FOO)
target_compile_definitions(foo PUBLIC -DFOO)  # -D removed
target_compile_definitions(foo PUBLIC "" FOO) # "" ignored
target_compile_definitions(foo PUBLIC -D FOO) # -D becomes "", then ignored
```

```cmake
target_compile_definitions(foo PUBLIC FOO=1)  # FOO's value is 1
```
