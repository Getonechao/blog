+++
title= "Amcl踩坑集"
description= "针对于amcl，工作中遇到的一些问题"
date= 2022-04-29T16:08:12+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "robat"
]

tags=  [
    "slam "," project"
]

+++

# Amcl踩坑集



## 2022.4.29

###### Q1

~~~shell
[ERROR] [1651219018.862852200]: Couldn't determine robot's pose associated with laser scan
[ WARN] [1651219018.942895300]: Failed to compute odom pose, skipping scan (Lookup would require extrapolation into the past.  Requested time 1651219008.851113400 but the earliest data is at time 1651219008.964298200, when looking up transform from frame [base_link] to frame [odom])
~~~

A2

~~~
原因是我在amcl中打印r2000激光雷达的数据，导致程序延迟，不能够及时的拿到r2000或者里程计的数据

源码：
amcl_node.cpp
// Where was the robot when this scan was taken?
  // pty:获得里程计数据
  pf_vector_t pose;
  if(!getOdomPose(latest_odom_pose_, pose.v[0], pose.v[1], pose.v[2],
                  laser_scan->header.stamp, base_frame_id_))
  {
    ROS_ERROR("Couldn't determine robot's pose associated with laser scan");
    return;
  }

~~~

