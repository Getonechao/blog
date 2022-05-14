+++
title= "4_ros常用命令"
description= "4_ros常用命令"
date= 2022-04-28T14:01:38+08:00
author= "somebody"
draft= true
image= "" 
math= true
categories= [
    "os"
]

tags=  [
    "ros ","robat "
]

+++

# ros常用命令



###### 创建ros工作空间

~~~bash
 mkdir -p ~/catkin_ws/src
 
 cd ~/catkin_ws
 
 catkin_make

 source devel/setup.bash
 
 echo $ROS_PACKAGE_PATH
 
 echo "source /home/chao/Desktop/code/02c/ros1/src/devel/setup.bash" >> ~/.bashrc 
~~~

###### 创建ros程序包

~~~bash
cd ~/catkin_ws/src

catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
~~~

###### ros参数命令

~~~shell

rosparam set     [param_name]       设置参数

rosparam get     [param_name]      获取参数

rosparam load    params.yaml       从文件中加载参数

rosparam dump 	 params.yaml  向文件中转储参数

rosparam delete  [param_name]      删除参数

rosparam list           列出参数名
~~~





