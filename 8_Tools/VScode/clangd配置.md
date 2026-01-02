# 一、clangd 配置内容

## clangd-v1.0

### setting 配置

```json
{
    // 自定义配置变量
    "DH_ENV": {
        "PROJECT_ROOT": "${workspaceFolder}",
		"BUILD_DIR": "${workspaceFolder}/build",
		"BIN_DIR": "${workspaceFolder}/build/bin",

		"PROJECT_NAME": "MyCppProject",
        "BUILD_TYPE": "Debug",
        "COMPILER_PATH": "D:/2_Binary/mingw64/bin/",
        "THREAD_COUNT": "4",
    },
    // 通用缩进设置
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "files.trimTrailingWhitespace": true,

    "cmake.cmakePath": "D:/Program Files/CMake/bin/cmake.exe",

    "C_Cpp.intelliSenseEngine": "disabled",
    "clangd.path": "D:/2_Binary/clangd/bin/clangd.exe",
    "clangd.arguments": [
        "-j=4",
        "--pretty",
        "--log=error",
        "--clang-tidy",
        "--enable-config",
        "--background-index",
        "--pch-storage=disk",
        "--header-insertion=never",
        "--completion-style=detailed",
        "--compile-commands-dir=build",
        "--query-driver=D:/2_Binary/mingw64/bin/g++.exe"
    ],
    "clangd.fallbackFlags": [
        //"-IE:\\VScode\\Test\\include"
    ],
}
```



### .clangd 配置

```yaml
Index:
  Background: Build

CompileFlags:
  Add: [-Wall, -std=c++11]
  Compiler: g++

```







# 二、clangd 配置说明

intelliSenseEngine 是必须要 disable 的，因为会与 clangd 的代码补全冲突的。

clangd arguments 的 `--query-driver` 是必须要有的，这里填写的是你的编译器的路径。它这个参数会根据你的编译器反推出需要的标准库头文件的位置，因此也就不需要我们去手动设置 C 与 C++标准库的路径了。不同的编译器用逗号隔开。

**--enable-config**：Read user and project configuration from YAML files.
Project config is from a .clangd file in the project directory.User config is from clangd/config.yaml in the following directories:

- Windows: `%USERPROFILE%\AppData\Local` 即 `C:\Users\Bob\AppData\Local\clangd\config.yaml`
- Mac OS: `~/Library/Preferences/`
- Others: `$XDG_CONFIG_HOME, usually ~/.config` 即 `~/.config/clangd/config.yaml`

vscode 的 clangd 插件有如下配置项：

- clangd.fallbackFlags: 设置头文件搜索路径
- clangd.path: clangd 的可执行文件路径
- clangd.arguments: clangd 服务运行时传递给可执行文件的参数
- clangd.detectExtensionConflicts: 设置 clangd 是否检测扩展的冲突
- clangd.serverCompletionRanking: 设置是否在键入时，对补全结果进行排序。
  clangd 的其他选项没有那么重要，这里就不一一列举。

---

Too many errors emitted, stopping now [clang: fatal_too_many_errors]: 

这个错误说明 **clangd 遇到了太多编译错误，直接停止了分析**。需要减少错误数量让它继续工作。

## 快速解决方案（按优先级）

### 1. **忽略无关错误**（最有效）
在项目根目录创建 `.clangd` 文件：
```yaml
Diagnostics:
  Suppress: ["*"]  # 临时抑制所有错误，先让补全工作

# 或只抑制特定错误
Diagnostics:
  Suppress:
    - "unknown*"
    - "include*"
    - "macro*"
```

### 2. **提高错误限制**
```yaml
CompileFlags:
  Add: ["-ferror-limit=1000"]  # 增加错误上限
```

### 3. **排除第三方/生成文件**
```yaml
CompileFlags:
  Add: [-w]  # 禁用所有警告

# 或者排除目录
Diagnostics:
  Suppress: ["./build/*", "./third_party/*"]
```

## 分步排查

### 步骤 1：找出主要错误源
```bash
# 手动编译查看真实错误
g++ -c your_file.cpp 2>&1 | head -20
```

### 步骤 2：修复关键错误
```yaml
# 临时解决方案：在 .clangd 中添加必要的定义
CompileFlags:
  Add:
    - "-D_MY_DEFINE=1"           # 缺少的宏定义
    - "-I./missing/include/path" # 缺少的头文件路径
    - "-std=c++17"               # 指定C++标准
```

### 步骤 3：验证配置
```bash
# 测试单个文件是否正常解析
clangd --check=main.cpp --compile-commands-dir=.
```

## 常见原因和解决

| 原因           | 解决方案                                 |
| -------------- | ---------------------------------------- |
| 缺少头文件路径 | 在 `.clangd` 中添加 `-I/path/to/include` |
| 宏定义缺失     | 添加 `-DMACRO=value`                     |
| C++标准不匹配  | 添加 `-std=c++11/14/17/20`               |
| 系统头文件问题 | 添加 `-isystemC:/mingw64/include`        |

## 紧急恢复方案
如果急需补全功能：
```yaml
# .clangd 配置（临时使用）
CompileFlags:
  Add: ["-w", "-ferror-limit=1000"]
Diagnostics:
  Suppress: ["*"]
```

**建议**：先用方案 1 让 clangd 工作起来，再逐步修复真实编译错误。