+++
title= "Bluez"
description= "linux官方蓝牙协议bluez的使用"
date= 2022-04-28T10:39:16+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    "c ","project"
]

+++

#  linux官方蓝牙协议bluez的使用

## 相关网址

- [官网源码](http://www.bluez.org/)
- [官网文档](http://wiki.bluez.org/wiki)
- [python调用bluez](https://github.com/IanHarvey/bluepy)
- [bluepy - a Bluetooth LE interface for Python — bluepy 0.9.11 documentation (ianharvey.github.io)](http://ianharvey.github.io/bluepy-doc/)

## 安装bluez

[(Ubuntu 20.04编译安装BlueZ-5.6_修不好的BUG的博客-CSDN博客_bluez安装](https://blog.csdn.net/qq_38529833/article/details/119777556)

[ 树莓派安装BlueZ协议栈（Raspberry pi Bluetooth LE）_PaulYoung_Blog的博客-CSDN博客_bluez安装](https://blog.csdn.net/talkxin/article/details/50609609?ops_request_misc=%7B%22request%5Fid%22%3A%22165112128316782388046008%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165112128316782388046008&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-50609609.142^v9^pc_search_result_control_group,157^v4^new_style&utm_term=安装bluez&spm=1018.2226.3001.4187)

~~~shell
xz -d bluez-5.64.tar.xz

tar zvf bluez-5.64.tar 

./configure --prefix=/usr/local --mandir=/usr/local/share/man --sysconfdir=/etc --localstatedir=/var

~~~

