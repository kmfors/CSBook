# 一、setting 配置内容

## setting-v1.0

```json
{
    // 自定义配置变量
    "DH_ENV": {
        "PROJECT_ROOT": "${workspaceFolder}/4_awesome/pulsar",
		"BUILD_DIR": "${workspaceFolder}/4_awesome/pulsar/build",
		"BIN_DIR": "${workspaceFolder}/4_awesome/pulsar/build",

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
    "cmake.ignoreCMakeListsMissing": true,

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
        "--compile-commands-dir=${workspaceFolder}/4_awesome/pulsar/build",
        "--query-driver=D:/2_Binary/mingw64/bin/g++.exe"
    ],
    "clangd.fallbackFlags": [
        //"-IE:\\VScode\\Test\\include"
    ],
    "clangd.inactiveRegions.useBackgroundHighlight": true,
    "editor.inlayHints.enabled": "offUnlessPressed",
}
```





# 二、setting 配置说明

"editor.inlayHints.enabled": "offUnlessPressed", 需要同时按住 ctrl与cmd