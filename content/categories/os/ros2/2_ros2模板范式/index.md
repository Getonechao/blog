+++
title= "2_ros2模板范式"
description= "文章简介"
date= 2022-09-03T17:13:39+08:00
author= "somebody"
draft= false
image= "" 
math= true
categories= [
    "boke"
]

tags=  [
    " "," "
]

+++

# ros2模板范式



## cpp

### 1. ament_cmake

set

~~~
#生成compile_command.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
#c编译器
set(CMAKE_C_COMPILER "/usr/bin/clang")
#cpp编译器
set(CMAKE_CXX_COMPILER "/usr/bin/clang++")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Werror")
~~~



固定范式

~~~
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

aux_source_directory(src SRC_LIST)
add_executable(${PROJECT_NAME} ${SRC_LIST} )
ament_target_dependencies(${PROJECT_NAME} rclcpp std_msgs)
~~~



安装

~~~
目录
install( DIRECTORY  XX DESTINATION XX)


可执行文件
install(TARGETS ${PROJECT_NAME} DESTINATION lib/${PROJECT_NAME})

~~~







## python