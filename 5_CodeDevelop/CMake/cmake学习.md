约定：

```cmake
# 泛指多个 item 的文件组成的源码列表
<sources>...

# 泛指多个 item 的文件组成的文件列表
<files>...

# 泛指多个 item 的目录组成的目录列表
<dirs>...
```
# 1. 基础项目的构建与编译

1、编写 CMakeList.txt 文件，构建系统的极简编写只需三行

```cmake
# 最小版本
cmake_minimum_require(VERSION 3.20)

# 项目名
project(demo)

# 生成可执行程序
add_executable(${PROJECT_NAME} demo.cpp)
```

2、生成构建系统

```shell
# 创建一个build目录，在build里生成makefile与其他文件
cmake -B build
```

3、编译项目

```shell
# 编译项目
cmake --build build
```

# 2. 指定 C++ 标准

cmake 提供了许多的预定义变量，各自都有着不同的作用意义。这些变量可以在 cmake 脚本中直接使用，以控制项目的构建过程。

其中许多变量以 `CMAKE_` 开头。在自己的项目中创建变量时要避免这种命名约定。

先学两个自变量，也是其中比较常用的。

- `CMAKE_CXX_STANDARD`：C++ 标准版本
- `CMAKE_CXX_STANDARD_REQUIRED`：是否为所需的最低版本

```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

确保将二者的声明添加到 `add_executable()` 的调用之上。

# 3. 添加版本编号、配置头文件

有时，在 CMakelists.txt 文件中定义的变量也可以在源代码中使用。在这种情况下，我们希望打印项目版本。

1、project 函数里指定项目名称、项目版本号

```cmake
project(demo VERSION 1.0)
```

2、将输入文件的内容拷贝到输出文件中，并将其 @VAR@ 变量替换为变量值。

```cmake
# 注意输入文件以 .h.i 结尾，生成的输出文件将以 .h 结尾
configure_file([xx_file.h.i] [xx_file.h])
```

输入文件内容：

```ini
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

- `<PROJECT-NAME>_VERSION_MAJOR`：项目主版本号（构建后自动生成的变量）
- `<PROJECT-NAME>_VERSION_MINOR`：项目次版本号（构建后自动生成的变量）

输出文件内容：

```c++
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR 0
```

3、将生成的输出文件（头文件）的目录添加到，目标文件依赖的头文件目录中

```cmake
target_include_directories(demo PUBLIC "${PROJECT_BINARY_DIR}")
```

> [!NOTE]
>
> - cmake 会以 CMakelists 文件中 `VAR` 名称作为匹配条件，对配置文件中的 `@VAR@` 形式的变量进行 `VAR` 的当前值替换。
> - `configure_file()` 命令默认（不指定路径）会将输出文件置于构建系统目录下。


# 4. 库文件的联编

我们可以用一个或多个子目录来组织项目，将所有源代码分散于各个子目录中，这样的子目录也称为库目录。而整个项目的构建与编译是需要库目录的共同联编。库目录不光存放着代码文件，也需要存放 CMakeLists 源文件来完成联编操作。

库目录的 CMakeLists 文件称为子CMakeLists 文件，项目工程的 CMakeLists 文件称为父CMakeLists 文件。

库目录的极简构建内容：

```cmake
add_library(<lib_file> [<type>] <sources>...)
```

工程目录极简构建的增量改动：

```cmake
# 为联编添加子目录
add_subdirectory(<lib_dir>)

# 目标文件依赖的库头文件路径
target_include_directories(<target> <lib_dirs>...)

# 目标文件依赖的库文件
target_link_libraries(<target> <lib_files>...)
```

一旦创建了库文件，就可以使用 `target_include_directories()` 和 `target_link_libraries()` 将其链接到我们的可执行目标。

> [!NOTE]
>
> - 建议将 `add_subdirectory()` 置于 `add_executable()` 之前

---

# 5. 添加编译开关

目标：添加编译开关的真实意图是，创建一个布尔变量的开关，用来启用或禁用源代码的各个部分。

库目录的原构建内容：

```cmake
add_library(mathLibs MathFunctions.cxx mysqrt.cxx)
```

具体场景：`mathLibs` 库是由 `MathFunctions.cxx` 与 `mysqrt.cxx` 代码文件编译生成的。现在我们不想使用 `mysqrt.cxx` 中的 `mysqrt` 函数，而使用标准库的 `sqrt` 函数。

期望：需要一种手段，能够只将有需要有用到的源代码参与库的编译，同时也提供其他源代码是否参与库编译的控制的手段。

- 步骤一：创建一个开关（布尔变量），由 `option()` 命令创建。如果发现该变量为真，就为目标文件添加一个编译定义，创建一个编译定义的命令是 `target_compile_definitions()`，创建出来的编译定义其实就是我们熟知的宏定义。
- 步骤二：先确定哪些源代码是需要参与到库编译的，而不确定的源代码则需要交由布尔变量开关来控制的，所以这个过程中需要对库的编译进行控制分离。

库目录的更改后的构建内容：

```cmake
add_library(mathLibs MathFunctions.cxx)

option(USE_MYMATH "Use tutorial provided math implementation" ON)

if(USE_MYMATH)
    target_compile_definitions(mathLibs PRIVATE "USE_MYMATH")
    # 单独生成一个新的库，并将其链接到最终的库
    add_library(sqrtLib STATIC mysqrt.cxx)
    target_link_libraries(mathLibs PRIVATE sqrtLib) 
endif()
```

注意：当 `USE_MYMATH` 开启时，编译定义 `USE_MYMATH` 将被设置。最终的目标文件里会有这特定的宏定义。而对应的源代码的相应修改也很简单，使用宏定义 `USE_MYMATH` 与 `ifdef` 语句来控制使用自定义的 `mysqrt` 函数还是标准库的 `sqrt` 函数。

---


# 6. 添加库的使用要求

意图：添加库的使用要求，减少耦合，减轻顶层 CMakeLists 负担，简化构建结构

目标：原先最终的 `<target>` 在链接依赖库时，也需要指定出依赖库的头文件路径。现在希望顶层 CMakeLists 文件中只链接依赖库的操作，而依赖库的头文件路径的指定从顶层 CMakeLists 文件转移至底层 CMakeLists 文件中。

底层 CMakeLists文件的增量改动:

```cmake
add_library(<lib_file> <sources>...)

target_include_directories(<lib_file> INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

底层 CMakeLists 文件的增量改动:

```cmake
# target_include_directories 注释掉依赖库头文件的路径指定
```

> [!TIP]
>
> - 关键字 `INTERFACE` 的填充意义：依赖库在编译时不需要指定其头文件路径，会在使用依赖库的时候指定头文件路径。



---

# 7. 用接口库设定 C++ 标准

意图：在目标在编译时添加自定义指定的编译要求（目标编译特性）

目标：添加 `INTERFACE` 库目标，来指定所需的 C++ 标准。

步骤 1：顶层 CMakeLists 文件中删除掉预定义变量的设值

```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

步骤 2：顶层 CMakeLists 文件中，创建一个名为 `<compile_flags>` 的 `INTERFACE` 库目标，并指定 `cxx_std_11` 为库目标的编译特性

```cmake
add_library(<compile_flags> INTERFACE)
target_compile_features(<compile_flags> INTERFACE cxx_std_11)
```

步骤 3：修改任意的 CMakeLists 文件，使得 `target_link_libraries()` 命令中依赖该 `<compile_flags>` 编译特性。

```cmake
target_link_libraries(<target> PUBLIC <compile_flags>...)
```



---

# 8. 如何理解编译标志

编译标志即 <compile_flags>

1、如何理解 <compile_flags> 要使用 `add_library()`命令生成？

```cmake
add_library(<compile_flags> INTERFACE)
```

一种自洽的解释：在使用普通的编译命令中，一般像 `g++ main.cpp -lLibs -Wall -O0` 这样的。如果将 <compile_flags> 也看做一个库目标，再结合限定作用标志，就可以很好像库目标一样作用于可执行目标文件。

对于 <compile_flags> 的使用，一般的限定作用标志为 `INTERFACE`，即仅在使用 <compile_flags> 时产生传播依赖，影响于其他目标。限定作用标志后为空的原因（自洽）：并不依赖于真正的文件，而是仅仅依赖自身的属性标志。

2、`target_compile_features()` 与 `target_compile_options()` 有什么区别呢？

`target_compile_features()` 和 `target_compile_options()` 都是 CMake 中用于设置目标编译选项的命令，区别是

- 目的不同
  - `target_compile_features()` 主要用于启用或禁用编译器特性，例如 C++11、C++14 等。
  - `target_compile_options()` 则用于直接向编译器传递命令行参数，这些参数可以是任何有效的编译器选项。
- 语法不同
  - `target_compile_features()` 接受一个特性名称作为参数，并可以指定要启用或禁用该特性。
  - `target_compile_options()` 接受一个或多个编译器选项作为参数，并将它们添加到目标的编译选项中。
- 作用范围不同
  - `target_compile_features()` 的作用范围仅限于指定的目标。
  - `target_compile_options()` 的作用范围可以是全局的（对所有目标生效）或者特定于某个目标。

总结一下，`target_compile_features()` 主要用于控制编译器特性的使用，而 `target_compile_options()` 则用于直接向编译器传递命令行参数。在实际项目中，根据需要选择合适的命令来设置编译选项。

所以大致理解 <compile_flags> 的使用，我们可以通过 <compile_flags> 控制编译器特性使用，或直接向编译器传递命令行参数（编译器选项）。


# 9. 添加编译器警告标志

意图：使用生成器表达式来添加编译器选项，从而作用于外部使用者。

目标：添加编译器警告标志

生成器表达式的一个常见用法是有条件地添加编译器标志，如语言级别或警告标志。一种不错的模式是将这些信息与 INTERFACE 目标相关联，从而使这些信息得以传播。

步骤 1：使用生成器表达式最低版本要求：

```cmake
cmake_minimum_required(VERSION 3.15)
```

步骤 2：使用 `COMPILE_LANG_AND_ID` 生成器表达式，如果当前语言与 language 一致，且编译器ID 在 compiler_ids 的列表中，则表达式值为 1，否则为 0。

```cmake
# CXX 语言，后跟多个编译器名称ID，统一为gcc_like_cxx变量
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")

# CXX 语言，后跟MSVC，统一为msvc_cxx变量
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
```

步骤 3：不管是 gcc_like_cxx 还是 msvc_cxx 变量值只能为 1 或 0。当使用 `target_compile_options()` 时，可以嵌套生成器表达式：

- `${gcc_like_cxx}` 值为 0，将取消编译标志选项，反之启用；
- `${msvc_cxx}` 值为 0，将取消编译标志选项，反之启用；

```cmake
target_compile_options(<compile_flags> INTERFACE
  "$<${gcc_like_cxx}:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>"
  "$<${msvc_cxx}:-W3>"
)
```

步骤 4：由于我们只想在构建过程中使用这些警告标志，由于已安装项目的用户而不应该继承我们的警告标志。为了明确这一点，因此将警告标志包装在一个生成器表达式中，并使用 `BUILD_INTERFACE` 条件来指定。

```cmake
target_compile_options(<compile_flags> INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
```

思考：解释一下最上面的一段话：`target_compile_options()` 使用的 `INTERFACE` 标志，将对使用该目标的其他目标产生传播依赖影响。而是用 `BUILD_INTERFACE` 条件的原因请看上面。

