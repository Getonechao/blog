+++
title= "1_ros2命令的使用"
description= "文章简介"
date= 2022-09-01T18:08:20+08:00
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



