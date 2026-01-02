# 一、tasks 配置内容

## tasks-v2.0

tasks-v2.0 版本添加了许多自定义的变量，而这些变量无法在原本 json 中定义，因此放在了 setting 文件中定义，搭配使用。

### setting.json

```json
{
    "DH_ENV": {
        "PROJECT_ROOT": "${workspaceFolder}",
		"BUILD_DIR": "${workspaceFolder}/build",
		"BIN_DIR": "${workspaceFolder}/build/bin",

		"PROJECT_NAME": "MyCppProject",
        "BUILD_TYPE": "Debug",
        "COMPILER_PATH": "D:/ProgramData/mingw64/bin",
        "THREAD_COUNT": "4",
    }
}
```



### tasks.json

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "CMake: configure",
            "type": "shell",
            "command": "cmake",
            "args": [
				"-S", "${config:DH_ENV.PROJECT_ROOT}",
				"-B", "${config:DH_ENV.BUILD_DIR}",
				"-G", "MinGW Makefiles",
				"-DCMAKE_BUILD_TYPE=${config:DH_ENV.BUILD_TYPE}",
				"-DCMAKE_EXPORT_COMPILE_COMMANDS=ON",
				"-DCMAKE_C_COMPILER=${config:DH_ENV.COMPILER_PATH}/gcc.exe",
				"-DCMAKE_CXX_COMPILER=${config:DH_ENV.COMPILER_PATH}/g++.exe"
			],
			"options": { "cwd": "${config:DH_ENV.PROJECT_ROOT}" },
			"group": {
                "kind": "build",
                "isDefault": true
            },
			"presentation": {
				"reveal": "always",
				"panel": "shared"
			},
            "problemMatcher": [ ],
            "detail": "CMake configuration task"
        },
        {
            "label": "CMake: build",
            "type": "shell",
            "command": "cmake",
            "args": [
                "--build", "${config:DH_ENV.BUILD_DIR}",
                "--config", "${config:DH_ENV.BUILD_TYPE}",
                "-j", "${config:DH_ENV.THREAD_COUNT}"
            ],
            "options": { "cwd": "${config:DH_ENV.PROJECT_ROOT}" },
            "group": {
                "kind": "build",
                "isDefault": false
            },
            "problemMatcher": [ "$msCompile" ],
			"presentation": {
				"reveal": "always",
				"panel": "shared"
			},
            //"dependsOn": "CMake: configure",
			"detail": "CMake build task"
        },
        {
            "label": "Run MyCppProject",
            "type": "shell",
            "command": "${config:DH_ENV.BIN_DIR}/${config:DH_ENV.PROJECT_NAME}.exe",
            "group": {
                "kind": "test",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "new"
            },
            "problemMatcher": []
        }
    ]
}
```



## tasks-v1.0

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "CMake: configure",
            "type": "shell",
            "command": "cmake",
            "args": [
				"-S", "${workspaceFolder}",
				"-B", "${workspaceFolder}/build",
				"-G", "MinGW Makefiles",
				"-DCMAKE_BUILD_TYPE=Debug",
				"-DCMAKE_EXPORT_COMPILE_COMMANDS=ON",
				"-DCMAKE_C_COMPILER=D:/ProgramData/mingw64/bin/gcc.exe",
				"-DCMAKE_CXX_COMPILER=D:/ProgramData/mingw64/bin/g++.exe"
			],
			"options": { "cwd": "${workspaceFolder}" },
			"group": "build",
			"presentation": {
				"reveal": "always",
				"panel": "shared"
			},
            "problemMatcher": [ ],
            "detail": "CMake configuration task"
        },
        {
            "label": "CMake: build",
            "type": "shell",
            "command": "cmake",
            "args": [
                "--build", "${workspaceFolder}/build",
                "--config", "Debug",
                "-j", "4"
            ],
            "options": { "cwd": "${workspaceFolder}" },
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": [ "$msCompile" ],
			"presentation": {
				"reveal": "always",
				"panel": "shared"
			},
            "dependsOn": "CMake: configure",
			"detail": "CMake build task"
        },
        {
            "label": "Run MyCppProject",
            "type": "shell",
            "command": "${workspaceFolder}/build/bin/MyCppProject.exe",
            "group": {
                "kind": "test",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "new"
            },
            "problemMatcher": []
        }
    ]
}
```







# 二、tasks 配置说明

## 创建过程

快捷键 `ctrl+shift+P` 打开命令面板，输入 `Tasks: Configure Task`
	
	"kind": "build" 归属于构建任务 命令为 Tasks: Run Build Task，默认快捷键 ctrl shift B
	
	"kind": "test" 归属于测试任务 命令为 Tasks: Run Test Task，可设置快捷键

注意：
  - 对于 Visual Studio 而言，编译 debug 还是 release，是需要在构建阶段指定的， 即 --config Debug
	对于 MinGW/GCC 而言，编译 debug 还是 release，是需要在配置阶段指定的， 即 -DCMAKE_BUILD_TYPE = Debug
	虽然前面用了 -DCMAKE_BUILD_TYPE = Debug ，后面又添加 --config Debug 但 CMake 会忽略这个参数（对 MinGW 无效）
	
  - "options": { "cwd": "${workspaceFolder}" } 让 CMake 命令在这个目录下执行
  
  - "group": "build" - 只把任务归到 build 分组
	"group": { "kind": "build", "isDefault": true } - 归到 build 分组并且设为默认构建任务
	"isDefault": true 让这个任务能通过快捷键 Ctrl+Shift+B 直接执行；否则还要选择执行哪个任务
	
  - "presentation": { // 控制任务运行时的终端面板，不填会默认
		"reveal": "silent",  // 后台运行，不打扰你
		"panel": "dedicated" // 如果有输出，会显示在专门的面板里
	}
	"reveal": 何时显示终端
		"silent": 静默运行，不自动弹出终端（推荐）
		"always": 总是弹出显示，默认
		"never": 从不显示
	"panel": 终端面板管理方式
		"dedicated": 为这个任务分配专用终端面板，不同任务输出分离
		"shared": 共享一个终端面板，默认
		"new": 每次都创建新终端
	
  - "dependsOn": "CMake: Configure Debug", 构建任务会自动依赖配置任务，先执行 configure，后执行 build
  
  - cmake 中 -S 用来指定项目里 根/顶级 CMakeLists.txt 所在的目录
	而 cwd 是命令执行的工作目录