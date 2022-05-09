+++
title= "6_launch文件"
description= "文章简介"
date= 2022-05-06T13:03:17+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "os"
]

tags=  [
    " ros","robat"
]

+++

# launch文件

## 模板

~~~xml
<?xml version="1.0"?>
<launch>
    
 	<include file="$(find pepperl_fuchs_r2000)/launch/r2000.launch"/>
 
 	<!--pkg:功能包名称(文件夹)  type：节点的可执行文件名称。name：运行节点名   -->
 	<node pkg="" type="" name="" output="screen"></node>
    
    
</launch>
~~~

