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

## 常用的tf坐标系

~~~
laser_link：激光雷达

base_link：车体

odom:里程计坐标系

map：单个机器人全局坐标系

earth：多个机器人协作
~~~

## ros坐标转换的类

~~~
rosmsg info geometry_msgs/TransformStamped
~~~

geometry_msgs头文件内容

~~~
Accel.h                      
AccelStamped.h                
AccelWithCovariance.h         
AccelWithCovarianceStamped.h  
Inertia.h
InertiaStamped.h
Point32.h
Point.h
PointStamped.h
Polygon.h
PolygonStamped.h
Pose2D.h
PoseArray.h
Pose.h
PoseStamped.h
PoseWithCovariance.h
PoseWithCovarianceStamped.h
Quaternion.h                
QuaternionStamped.h          
Transform.h                  
TransformStamped.h     
Twist.h                
TwistStamped.h        
TwistWithCovariance.h
TwistWithCovarianceStamped.h  
Vector3.h
Vector3Stamped.h
Wrench.h
WrenchStamped.h
~~~





###  1. geometry_msgs::TransformStamped

坐标系之间的关联信息

~~~
std_msgs/Header header
  uint32 seq
  time stamp
  string frame_id
string child_frame_id
geometry_msgs/Transform transform
  geometry_msgs/Vector3 translation
    float64 x
    float64 y
    float64 z
  geometry_msgs/Quaternion rotation
    float64 x
    float64 y
    float64 z
    float64 w
~~~

### 2.geometry_msgs/PointStamped 

坐标点信息

~~~
std_msgs/Header header
  uint32 seq
  time stamp
  string frame_id
geometry_msgs/Point point
  float64 x
  float64 y
  float64 z
~~~

