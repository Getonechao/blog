+++
title= "如何安装anaconda"
description= "文章简介"
date= 2022-07-02T12:05:57+08:00
author= "somebody"
draft= true
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    " python"
]

+++

# 安装anaconda

## 1.环境配置

~~~bash
D:\[Anaconda]\ 
D:\Anaconda\Scripts 
D:\Anaconda\Library\bin 
D:\Anaconda\Library\mingw-w64\bin（可选）
~~~

## 2. 换源

打开C:\Users\Geton\.condarc

~~~
channels:
  - defaults
show_channel_urls: true
default_channels:
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
~~~

## 3.创建新的虚拟环境

~~~
conda create -p path python=3.10
~~~

