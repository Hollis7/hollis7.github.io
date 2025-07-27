---
title: c primer ch1
categories:
- c
tags:
- c
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括vscode一些配置，以及尝试运行、调试程序

<!--more-->

### launch.json

~~~json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Run/Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/bin/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "build"
        }
    ]
}
~~~

### task.json

~~~json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "cmake",
            "type": "shell",
            "command": "cmake",
            "args": [
                "-DCMAKE_BUILD_TYPE=Debug",
                "-S",
                "${workspaceFolder}",
                "-B",
                "${workspaceFolder}/build"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": []
        },
        {
            "label": "make",
            "type": "shell",
            "command": "make",
            "args": [
                "-C",
                "${workspaceFolder}/build",
                "-j4"  // 并行编译加快速度
            ],
            "problemMatcher": []
        },
        {
            "label": "build",
            "dependsOrder": "sequence",  // 确保 cmake 先执行，make 后执行
            "dependsOn": ["cmake", "make"],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": []
        }
    ]
}
~~~

### CmakeLists.txt

~~~cmake
cmake_minimum_required(VERSION 3.10)
project(c_primer_6 C)

# 设置C标准
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# 调试模式下的编译选项
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_options(-g -O0)  # -g 生成调试信息，-O0 禁用优化
endif()

# 设置输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# 包含头文件目录
include_directories(include)

# 查找src/ch1目录下的所有.c文件
file(GLOB CH1_SOURCES "src/ch1/*.c")

# 查找src/ch2目录下的所有.c文件
file(GLOB CH2_SOURCES "src/ch2/*.c")

# 为每个源文件创建可执行文件
foreach(source ${CH1_SOURCES})
    get_filename_component(executable ${source} NAME_WE)
    add_executable(${executable} ${source})
endforeach()

foreach(source ${CH2_SOURCES})
    get_filename_component(executable ${source} NAME_WE)
    add_executable(${executable} ${source})
endforeach()

# 如果你有库文件需要链接，可以这样添加
# target_link_libraries(hello mylib)
~~~

### settings.json

~~~json
{
    "files.associations": {
        "stdio.h": "c"
    }
}
~~~

### 文件结构

~~~bash
hollis@hollis7:~/cProject/c_primer_6$ tree -L 3 ../c_primer_6
../c_primer_6
├── CMakeLists.txt
├── README.md
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   │   ├── 2_1_first.dir
│   │   ├── 2_1_print_name.dir
│   │   ├── 2_2_fathm_ft.dir
│   │   ├── 2_3_two_func.dir
│   │   ├── 3.22.1
│   │   ├── CMakeDirectoryInformation.cmake
│   │   ├── CMakeOutput.log
│   │   ├── CMakeTmp
│   │   ├── Makefile.cmake
│   │   ├── Makefile2
│   │   ├── TargetDirectories.txt
│   │   ├── cmake.check_cache
│   │   ├── ex1.dir
│   │   ├── hello.dir
│   │   └── progress.marks
│   ├── Makefile
│   ├── bin
│   └── cmake_install.cmake
├── include
│   └── mylib
├── lib
└── src
    ├── ch1
    │   ├── ex1.c
    │   └── hello.c
    └── ch2
        ├── 2_1_first.c
        ├── 2_2_fathm_ft.c
        └── 2_3_two_func.c

17 directories, 17 files
~~~

