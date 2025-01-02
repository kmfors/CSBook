# 信息变量

提供信息的变量

- 
- PROJECT_BINARY_DIR：项目构建系统的目录，也是二进制目标文件的生成路径。
- PROJECT_NAME：`project()` 命令中给定的项目名
- PROJECT_SOURCE_DIR：在当前目录或父目录下，最后一次显式调用 `project()` 命令的 CMakeList 文件所在的源目录。

```shell
# 在build目录下，调用父目录下CMakeLists 文件中的 project()命令
cmake ..

# 在当前目录下，调用当前目录下CMakeLists 文件中的 project()命令
cmake .
```
  

解释：项目的构建系统的目录，也是二进制目标文件的生成路径。

# 行为变量

改变行为的变量

# 系统描述变量


# 控制构建变量


# 程序语言变量


