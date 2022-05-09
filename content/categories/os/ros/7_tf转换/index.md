+++
title= "7_tf转换"
description= "ros tf坐标系统"
date= 2022-05-04T15:13:09+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "os"
]

tags=  [
    " ros","robat "
]

+++

# TF转换

## 安装

~~~shell
sudo apt install ros-melodic-turtle-tf
~~~

## 可视化观测tf tree

~~~shell
#pdf
rosrun tf view_frames

#stdout
rosrun tf tf_echo node1 nodde2
~~~

