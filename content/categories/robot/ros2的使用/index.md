+++
title= "ros2:命令的使用"
description= "文章简介"
date= 2022-09-01T18:08:20+08:00
author= "somebody"
draft= false
image= "" 
math= true
categories= [
    "robot"
]

tags=  [
    " robot","ros2 "
]

+++

# ros2命令



## ros2 pkg 创建功能包

~~~
# cpp
ros2 pkg create  --dependencies rclcpp std_msgs --build-type ament_cmake  [pkg name]
# python
ros2 pkg create  --dependencies rclpy std_msgs --build-type ament_python  [pkg name]

~~~







## colcon--编译

~~~
#下载依赖
rosdep install -y --from-paths src --rosdistro $ROS_DISTRO

colcon build --symlink-install 

------------------
--symlink-install : build目录中库文件软连接到install目录
--cmake-args：      cmake编译
--packages-select： 制定编译某个包
--parallel-workers （NUMBER）：要并行处理的最大作业数， 默认值是逻辑 CPU 内核数

~~~



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