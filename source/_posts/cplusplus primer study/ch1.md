---
title: c++ primer ch1
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括vscode一些配置，以及尝试运行、调试程序

<!--more-->

### 配置vscode

#### launch.json

~~~json
{
    "version": "0.2.0",
    "configurations": [
        {
           	...
            "program": "${workspaceFolder}/ch1/build/exercise1",
           	...
            "miDebuggerPath": "/usr/bin/gdb",
        }
    ]
}
~~~

主要是配置好编译好的可执行程序位置以及调试工具地址

~~~bash
/home/hollis/cProject/c_plus_study
g++ -g exercise1.cpp -o build/exercise1
~~~

#### task.json

~~~json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "C/C++: g++ 生成活动文件",
			"command": "/usr/bin/g++",
			"args": [
				"-fdiagnostics-color=always",
				"-g",
				"${file}",
				"-o",
				// "${fileDirname}/${fileBasenameNoExtension}"
				"${fileDirname}/build/${fileBasenameNoExtension}"
			],
			"options": {
				"cwd": "${fileDirname}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": {
				"kind": "build",
				"isDefault": true
			},
			"detail": "编译器: /usr/bin/g++"
		}
	]
}
~~~

这样应用程序就会自动编译到`build`文件下

### 文件结束符

当从键盘向程序输入数据时，对于如何指出文件结束，不同操作系统有不同的约定。在Windows系统中，输入文件结束符的方法是敲`Ctrl+Z`(按住Ctrl键的同时按Z键)，然后按Enter或Return键。在UNIX系统中，包括Mac OS X系统中，文件结束符输入是用`Ctrl+D`。

~~~c++
#include <iostream>

int main()
{
	int sum = 0, value = 0;

	while (std::cin >> value)
	{
		sum += value;
	}

	std::cout << sum << std::endl;

	return 0;
}
~~~

### cmake管理项目

初步管理，主要是为了管理自己写的头文件和一些路径

#### CMakeLists.txt（AI编写）

~~~cmake
cmake_minimum_required(VERSION 3.10)  # 最低 CMake 版本要求
project(C_PLUS_STUDY)                    # 项目名称

# 设置 C++ 标准（例如 C++11/14/17/20）
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置调试编译选项（Debug 模式）
set(CMAKE_BUILD_TYPE Debug)  # 启用调试符号

# 头文件路径变量
set(MYLIB_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include/mylib)

# 添加可执行文件
add_executable(exercise1_21           # 目标名称（可执行文件名）
    src/ch1/exercise1_21.cpp          # 源文件路径
)
add_executable(exercise1_22
    src/ch1/exercise1_22.cpp
)

# 设置头文件搜索路径
# 批量设置头文件路径
foreach(target exercise1_21 exercise1_22)
    target_include_directories(${target} PRIVATE ${MYLIB_INCLUDE_DIR})
endforeach()

# 可选：设置输出目录（默认在 build/ 下）
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)

# 可选：显式指定调试编译选项
target_compile_options(exercise1_22 PRIVATE -g -O0)  # -g 生成调试符号，-O0 禁用优化
~~~

编译代码如下：

~~~bash
mkdir build && cd build
cmake ..
cmake --build .
./bin/exercise1_20
~~~

### cmake项目调试

#### task.json

~~~json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "cmake-build",
            "type": "shell",
            "command": "cd build && cmake --build .",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
~~~

#### launch.json

~~~json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/bin/exercise1_22",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "cmake-build" //可选：关联编译任务（需配置 tasks.json）
        }
    ]
}
~~~

#### 简化launch.json

~~~json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug mode",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/bin/exercise1_18",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "cmake-build"  //可选：关联编译任务（需配置 tasks.json）
        }
    ]
}
~~~

