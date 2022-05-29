+++
title= "Vscode的c_cpp配置"
description= "文章简介"
date= 2022-05-29T10:45:50+08:00
author= "somebody"
draft= true
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    " vscode"
]

+++

# .vscode的配置



## c_cpp_properties.json



ctrl+shift+p ,运行`C/Cpp: Edit configurations...`

~~~json

{
    "configurations": [
        {
            "name": "linux",
            "includePath": [
                "/opt/ros/melodic/include",
                "/usr/include",
                "${workspaceFolder}/**",
                "${workspaceFolder}/devel/include"
            ],
            "intelliSenseMode": "linux-gcc-x64",
            "compilerPath": "/usr/bin/gcc",
            "cppStandard": "c++17",
            "cStandard": "c17"
        }
    ],
    "version": 4
}
~~~

## tasks.json

ctrl+shift+p ,运行task

~~~json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        /*1. mkdir build*/
        {
            "label": "mkdir build",
            "type": "shell",
            "command": "mkdir",
            "args": [
                "-p",
                "build"
            ],
            "options": {"cwd": "${workspaceFolder}"},
            "group": "build",
        },
        /*2. cmake ..*/
        {
            "label": "cmake ..",
            "type": "shell",
            "command": "cmake",
            "args": [
                ".."
            ],
            "options": {
                "cwd": "${workspaceFolder}/build",
                },
            "group": "build",
            "dependsOn":[
                "mkdir build",//表示在"创建build"任务结束后进行
            ]
        },
        /*3. make */
        {
            "label": "make",
            "type": "shell",
            "command": "make",
            "args": [
                ""
            ],
            "group": "build",
            "dependsOn":[
                "cmake ..",//表示在"创建build"任务结束后进行
            ],
            "presentation": {//配置用于显示任务输出并读取其输入的面板
                "echo": true,
                "reveal": "never",
                "focus": false,
                "panel": "shared",
                "showReuseMessage": true,
                "clear": false
            },
            "options": {"cwd": "${workspaceFolder}/build"},
        }
    ]
}
~~~

预定义变量名字

~~~
${workspaceFolder} - 当前工作目录(根目录)
${workspaceFolderBasename} - 当前文件的父目录
${file} - 当前打开的文件名(完整路径)
${relativeFile} - 当前根目录到当前打开文件的相对路径(包括文件名)
${relativeFileDirname} - 当前根目录到当前打开文件的相对路径(不包括文件名)
${fileBasename} - 当前打开的文件名(包括扩展名)
${fileBasenameNoExtension} - 当前打开的文件名(不包括扩展名)
${fileDirname} - 当前打开文件的目录
${fileExtname} - 当前打开文件的扩展名
${cwd} - 启动时task工作的目录
${lineNumber} - 当前激活文件所选行
${selectedText} - 当前激活文件中所选择的文本
${execPath} - vscode执行文件所在的目录
${defaultBuildTask} - 默认编译任务(build task)的名字
~~~

