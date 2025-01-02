生成器表达式在生成构建系统时进行评估，以生成每个构建配置的特定信息。它们的格式为 `$<...>`。例如

```cmake
target_include_directories(tgt PRIVATE /opt/include/$<CXX_COMPILER_ID>)
```

根据所使用的 C++ 编译器，这将扩展为 `/opt/include/GNU`、`/opt/include/Clang` 等。

在许多目标属性（如 LINK_LIBRARIES、INCLUDE_DIRECTORIES、COMPILE_DEFINITIONS 等）的上下文中都允许使用生成器表达式。在使用 target_link_libraries()、target_include_directories()、target_compile_definitions() 等命令填充这些属性时，也可以使用它们。它们可以实现有条件链接、编译时使用的有条件定义、有条件包含目录等功能。这些条件可以基于编译配置、目标属性、平台信息或任何其他可查询的信息。

make 构建分为 config 和 build 阶段，生成器表达式的值在 build 阶段才得到。生成器表达式可用于根据某些条件设置某些变量和编译选项

配置构建系统，构建目标。

1、CMakeLists.txt 解析过程

CMake 构建过程分为两个阶段

- 配置阶段，CMake 会读取项目的 CMakeLists.txt 文件，并根据其中的指令和参数来生成 Makefile 或者 IDE 的项目文件
    - 检查编译器和工具链是否可用，并设置编译器选项和链接选项
    - 检查系统库和第三方库是否可用，并设置库的路径和链接选项
    - 检查项目的源代码文件，并设置编译选项和链接选项
    - 生成 Makefile 或者 IDE 的项目文件
    - 根据不同的平台和编译器生成不同的 Makefile 或者项目文件，以保证项目可以在不同的平台和编译器上构建
- 构建阶段，CMake 会根据配置阶段生成的 Makefile 或者项目文件来执行实际的构建操作
    - 根据 Makefile 或者项目文件中的指令和参数来编译源代码文件，并生成目标文件
    - 根据 Makefile 或者项目文件中的指令和参数来链接目标文件，并生成可执行文件或者库文件

2、生成器表达式

CMake 生成器表达式是一种特殊的语法，用于在 CMake 构建系统中动态地生成构建规则。它们可以用于指定编译器选项、链接选项等。

本节先学习其中几种表达式：

- `$<condition:true_string>`
    - 如果 condition 为 1，则此表达式结果为 true_string
    - 如果 condition 为 0，则此表达式结果为空
- `$<IF:condition,true_string,false_string>`
    - 如果 condition 为 1，则此表达式结果为 true_string
    - 如果 condition 为 0，则此表达式结果为 false_string
- `$<BOOL:string>`
    - 根据 if 中对字符串的判断，将字符串转为 bool 值。
- `$<AND:conditions>`
    - 逻辑与，将多个判断条件使用,分隔开，然后对这些判断条件进行与运算。
- `$<OR:conditions>`
    - 逻辑或，将多个判断条件使用,分隔开，然后对这些判断条件进行或运算。
- `$<NOT:condition>`
    - 逻辑反，将多个判断条件使用,分隔开，然后对这些判断条件进行取反运算。
- `$<COMPILE_LANG_AND_ID:language,compiler_ids>`
    - 如果当前所用的语言与 language 一致且编译器 ID 在 compiler_ids 的列表中，则表达式值为 1，否则为 0
    - language 值主要为 CXX 和 C
    - compiler_ids 主要有 GNU、Clang、MSVC 等，有多个时用逗号隔开

生成器表达式因为是在生成阶段可用，所以不能在配置阶段进行输出，可用下面方式进行查看：

```cmake
add_custom_target(ged COMMAND ${CMAKE_COMMAND} -E echo "$<1:hello>")
```

- `${CMAKE_COMMAND}` 是一个变量，它包含了 CMake 命令的完整路径，你可以用它来调用 CMake 命令。
- `E` 选项提供了一些平台无关的基本命令。
- 使用 `${CMAKE_COMMAND} -E` 是在不同平台间移植 Shell 命令的一种常用方法，可以避免平台特定的命令差异。

配置完成之后，用以下命令进行输出

```cmake
cmake --build . --target ged

# 用make可简写为下面的命令
make ged
```

但不是所有的表达式都能这样输出，有的表达式无法输出，比如 `$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>`


![[20240914095839.png]]


Generator Expressions：在构建系统生成过程中进行评估，以生成每个构建配置的特定信息。

在许多目标属性（如 LINK_LIBRARIES、INCLUDE_DIRECTORIES、COMPILE_DEFINITIONS 等）的上下文中，都允许使用生成器表达式。在使用 target_link_libraries()、target_include_directories()、target_compile_definitions() 等命令填充这些属性时，也可以使用它们。

生成器表达式可用于启用条件链接、编译时使用的条件定义、条件包含目录等。这些条件可以基于编译配置、目标属性、平台信息或任何其他可查询的信息。

生成器表达式有多种类型，包括逻辑表达式、信息表达式和输出表达式。

逻辑表达式用于创建条件输出。基本表达式是 0 和 1 表达式。 `$<0:...>` 的结果是空字符串，`$<1:...>` 的结果是 .... 的内容。它们还可以嵌套。