+++
title= "Ros学习之旅3"
description= "如何安装ros"
date= 2022-04-01T13:25:02+08:00
author= "chao"
draft= false
image= "" 
categories= [
    "os"
]

tags=  [
    "ros"
]
+++

# 如何安装ros

## 脚本安装
~~~
wget http://fishros.com/install -O fishros && sudo bash fishros
~~~
[原文章链接：鱼香ros](https://mp.weixin.qq.com/s/8hTrKL0N5y9i6s9ujhp0UA)

## rosdep 安装

> note: 注意安装ros过程中，可以不安装rosdep，它不是ros系统必须安装的，它的功能类似于ubuntu中的apt
>
> ，当我们安装ros的一些功能包的时候，也可以用apt安装，可以不用rosdep

[参考视频：小鱼在古月居开课视频--聊聊ROS安装过程中的那些坑](https://class.guyuehome.com/detail/p_61c588e1e4b0219857fdb40e/6)