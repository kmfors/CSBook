# 规范的 makefile

```Makefile
# 1、编译器
CXX := g++
# 2、编译选项
CXXFLAGS := -g -O0 -std=c++11 -Wall

# 源文件路径
PATH_SRC := src
# 目标文件路径
PATH_OBJ := obj
# 可执行程序路径
PATH_TEST := test

# 将获取的源文件列表匹配生成对应的目标文件列表
OBJ_FILES := $(patsubst $(PATH_SRC)/%.cpp, $(PATH_OBJ)/%.o, $(wildcard $(PATH_SRC)/*.cpp))

# 可执行程序的名称
TARGET := $(TEST_DIR)/my_program

# 默认目标
all: $(TARGET)

# 生成可执行程序：obj->target
$(TARGET): $(OBJ_FILES)
	$(CXX) $(CXXFLAGS) $^ -o $@

# 编译生成目标文件
$(PATH_OBJ)/%.o: $(PATH_SRC)/%.cpp | $(PATH_OBJ)
	$(CXX) $(CXXFLAGS) -c $< -o $@

# 创建目标文件夹
$(PATH_OBJ):
	mkdir -p $(PATH_OBJ)

# 清理规则
clean:
	rm -rf $(OBJ_DIR) $(TARGET)

# 声明伪目标，避免与同名文件冲突
.PHONY: all clean
```

这个 Makefile 是一个简单的 C++ 项目构建脚本，用于编译源代码并生成可执行程序。让我来解释每个部分的作用：

1. **编译器和编译选项定义**：
   - `CXX := g++`：定义了 C++ 编译器为 `g++`。
   - `CXXFLAGS := -g -O0 -std=c++11 -Wall`：定义了编译选项，包括开启调试信息 (`-g`)、禁用优化 (`-O0`)、使用 C++11 标准 (`-std=c++11`) 和开启警告信息 (`-Wall`)。

2. **路径定义**：
   - `PATH_SRC := src`：源文件路径为 `src` 目录。
   - `PATH_OBJ := obj`：目标文件路径为 `obj` 目录。
   - `PATH_TEST := test`：可执行程序路径为 `test` 目录。

3. **生成目标文件列表**：
   - `OBJ_FILES := $(patsubst $(PATH_SRC)/%.cpp, $(PATH_OBJ)/%.o, $(wildcard $(PATH_SRC)/*.cpp))`：这行代码将源文件列表中的每个 `.cpp` 文件路径替换为相应的 `.o` 目标文件路径，并将结果存储在 `OBJ_FILES` 变量中。

4. **可执行程序名称定义**：
   - `TARGET := $(TEST_DIR)/my_program`：定义了可执行程序的名称为 `test/my_program`。

5. **默认目标**：
   - `all: $(TARGET)`：默认目标为 `all`，它依赖于 `$(TARGET)`，表示构建可执行程序。

6. **生成可执行程序规则**：
   - `$(TARGET): $(OBJ_FILES)`：`$(TARGET)` 的依赖是所有的目标文件 (`$(OBJ_FILES)`)。
   - `$(CXX) $(CXXFLAGS) $^ -o $@`：使用编译器 (`$(CXX)`) 编译所有目标文件 (`$^`) 并链接生成可执行程序 (`$@`)。

7. **编译生成目标文件规则**：
   - `$(PATH_OBJ)/%.o: $(PATH_SRC)/%.cpp | $(PATH_OBJ)`：定义了将每个 `.cpp` 源文件编译为相应的 `.o` 目标文件的规则。（ `|` 前置条件，表示在执行规则之前，需要先创建 $(PATH_OBJ) 目录。如果目标文件夹不存在，则会先创建它）
   - `$(CXX) $(CXXFLAGS) -c $< -o $@`：使用编译器 (`$(CXX)`) 编译指定的源文件 (`$<`) 并将结果输出到指定的目标文件 (`$@`)。


8. **创建目标文件夹**：
   - `$(PATH_OBJ):`：定义了生成目标文件夹的规则。
   - `mkdir -p $(PATH_OBJ)`：创建目标文件夹。

9. **清理规则**：
   - `clean:`：定义了清理规则，用于删除生成的目标文件和可执行程序。
   - `rm -rf $(OBJ_DIR) $(TARGET)`：删除目标文件夹和可执行程序。

10. **伪目标声明**：
    - `.PHONY: all clean`：声明了 `all` 和 `clean` 为伪目标，以确保 Make 不会误将它们当作文件名处理。

总的来说，这个 Makefile 的作用是根据指定的源代码文件路径，编译所有的 C++ 源文件，并链接生成一个可执行程序，同时提供了清理规则以清除生成的目标文件和可执行程序。
