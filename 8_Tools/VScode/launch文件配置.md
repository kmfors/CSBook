# 一、launch 配置内容

## launch-v1.0

launch.json 文件

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/bin/MyCppProject.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:/ProgramData/mingw64/bin/gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "CMake: build",
        }
    ]
}
```



# 二、launch 配置说明

 定义调试会话的设置、指定如何启动和调试程序、包含调试器配置、程序路径、启动参数等

	点击左侧活动栏的 "Run and Debug" 图标
	
	点击创建一个 launch.json 文件
	
	选择 C++ (GDB/LLDB) 作为环境
	
	选择 (gdb) Launch 生成默认配置
	
	更改 miDebuggerPath 字段值
	
	"externalConsole": false 使用集成终端，true 使用独立终端