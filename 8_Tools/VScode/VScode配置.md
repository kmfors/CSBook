# clangd代码补全

vscode有个clangd插件，我们在使用时需要做一下配置。

vscode的clangd插件有如下配置项：
- clangd.fallbackFlags: 设置头文件搜索路径
- clangd.path: clangd的可执行文件路径
- clangd.arguments: clangd服务运行时传递给可执行文件的参数
- clangd.detectExtensionConflicts: 设置clangd是否检测扩展的冲突
- clangd.serverCompletionRanking: 设置是否在键入时，对补全结果进行排序。
clangd的其他选项没有那么重要，这里就不一一列举。

# clangd.arguments

```json
{
    "C_Cpp.intelliSenseEngine": "disabled",
    "clangd.path": "C:\\clangd_17\\bin\\clangd.exe",
    "clangd.arguments": [
        // 全局补全（会自动补充头文件）
        //"--all-scopes-completion",  // 还是手动配置好一些
        // 在后台自动分析文件（基于complie_commands)
        "--background-index",
        // 静态代码分析工具
        "--clang-tidy",
        // 查找编译命令文件,标记compelie_commands.json文件的目录位置
        "--compile-commands-dir=build",
        // 更详细的补全内容,完成风格
        "--completion-style=detailed",
        // 启用配置文件
        "--enable-config",
        // 补充头文件的形式
        "--header-insertion=never", //iwyu
        // 日志级别
        "--log=error",
        // 同时开启的任务数量
        "-j=4",
        // 偏移量编码
        "--offset-encoding=utf-8",
        // 预编译头文件被存储在磁盘上，并在需要时加载到内存中使用
        "--pch-storage=disk",
        // 格式化输出
        "--pretty",
        "--query-driver=C:\\msys64\\mingw64\\bin\\g++.exe",
//----------------------------------------------------------------------------

        "--check[=<string>]",  // 0，要应用的代码检查器或规则
        // 当没有找到`.clang-format`文件时，默认应用的`clang-format`样式通常称为“默认风格”
        "--fallback-style",
        // 指定函数参数占位符的功能
        "--function-arg-placeholders",
        // 在自动生成的头文件中插入装饰符(标记特定区域的注释或者指令)
        "--header-insertion-decorators",
        // 自动导入检测
        "--import-insertions",
        // 客户端路径与服务端路径的转换
        "--path-mappings=<string>",
        // 指定项目的根目录
        "--project-root=<string>",
        "--remote-index-address=<string>",
        "--rename-file-limit=<int>",
        "--version"        
    ],
}
```

intelliSenseEngine是必须要disable的，因为会与clangd的代码补全冲突的。

clangd arguments的 `--query-driver` 是必须要有的，这里填写的是你的编译器的路径。它这个参数会根据你的编译器反推出需要的标准库头文件的位置，因此也就不需要我们去手动设置C与C++标准库的路径了。不同的编译器用逗号隔开。

**--enable-config**：Read user and project configuration from YAML files.
Project config is from a .clangd file in the project directory.User config is from clangd/config.yaml in the following directories:
- Windows: `%USERPROFILE%\AppData\Local` 即 `C:\Users\Bob\AppData\Local\clangd\config.yaml`
- Mac OS: `~/Library/Preferences/`
- Others: `$XDG_CONFIG_HOME, usually ~/.config` 即 `~/.config/clangd/config.yaml`


# MSYS2配置

msys2在下载好的所需的编译器与工具后，必做的一个操作是设置环境变量，不设置环境变量的话，clangd找不到你想要将其引导的标准库头文件的位置。

```shell
#cmake g++路径
F:\msys64\mingw64\bin

# make路径
F:\msys64\usr\bin
```


# clangd.fallbackFlags

用于设置头文件搜索路径

```json
{
    "clangd.fallbackFlags": [
        "-IE:\\VScode\\Test\\include"
    ],
}
```


# config.yaml

这一步也是必须的，

```yaml
Index:
  Background: Build

CompileFlags:
  Add: [-Wall, -std=c++11]
  Compiler: g++

```

如果不去指定编译器的类型的话，默认使用的是clang编译器，现在我们指定了g++的编译器。

# 调试

因为装了C/C++的插件。所以直接左侧的bug昆虫图标，创建一个launch.json文件。

aunch.json用于运行.exe文件，tasks.json则是被指定的运行前的自动任务，它将.cpp文件编译成.exe文件，而c_cpp_properties.json则是扩展C/C++ Extension Pack中的一些设置。

# 注意：

"C_Cpp.intelliSenseEngine": "disabled" 这会导致自带的调试与运行按钮不会出现在右上角，除非把这一句去掉才可以，但带来的坏处是与clangd的智能感知发生冲突。