+++
title= "Libmodbus"
description= "使用libmodbus"
date= 2022-04-20T00:18:44+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    " project","c"
]

+++

# libmodbus的使用

## 安装

~~~
wget https://github.com/stephane/libmodbus/archive/refs/tags/v3.1.7.zip

unzip v3.1.7.zip

sudo apt install libtool

sudo apt install autoconf

./autoconf

./configure --prefix=/usr/local/

sudo make install
~~~

## 源码解析

参考网址

- [【嵌入式】Libmodbus源码分析(一)-类型和结构体_沧海一笑-dj的博客-CSDN博客_libmodbus源码](https://blog.csdn.net/dengjin20104042056/article/details/116669466?ops_request_misc=%7B%22request%5Fid%22%3A%22165038752016781685380520%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165038752016781685380520&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-116669466.142^v9^pc_search_result_control_group,157^v4^new_style&utm_term=modbus_mapping_t&spm=1018.2226.3001.4187) 
- [【嵌入式】Libmodbus源码分析(二)-常用接口函数分析_沧海一笑-dj的博客-CSDN博客_modbus接口函数](https://dengjin.blog.csdn.net/article/details/116721415)  
- [【嵌入式】Libmodbus源码分析(三)-modbus相关函数分析_沧海一笑-dj的博客-CSDN博客_libmodbus代码示例](https://dengjin.blog.csdn.net/article/details/116752449)
- [【嵌入式】Libmodbus源码分析(四)-RTU相关函数分析_沧海一笑-dj的博客-CSDN博客](https://dengjin.blog.csdn.net/article/details/116752863)
- [【嵌入式】Libmodbus源码分析(五)-TCP相关函数分析_沧海一笑-dj的博客-CSDN博客](https://dengjin.blog.csdn.net/article/details/116753916)
- [【嵌入式】嵌入式天地博客汇总_沧海一笑-dj的博客-CSDN博客](https://dengjin.blog.csdn.net/article/details/116999754)
- [libmodbus官方手册中文翻译_跃动的风的博客-CSDN博客_libmodbus使用说明](https://blog.csdn.net/qq_23670601/article/details/82155378?ops_request_misc=&request_id=&biz_id=102&utm_term=libmodbus debug&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-82155378.142^v9^pc_search_result_control_group,157^v4^new_style&spm=1018.2226.3001.4187)