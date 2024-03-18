+++
title= "Raspberry：Wiringpi的安装及使用"
description= "Wiringpi的安装及使用"
date= 2022-05-04T14:01:09+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "os"
]

tags=  [
    " raspberry"
]

+++

# WiringPi

## 安装

[WiringPi](http://wiringpi.com/)

###### 官网介绍截取

1. ***WiringPi*** is a ***PIN*** based GPIO access library written in C for the BCM2835, BCM2836 and BCM2837 SoC devices used in all **Raspberry Pi.** versions. The source code is not publicly available but may be made available to those who wish commercial support.

2. It’s designed to be familiar to people who have used the Arduino “*wiring*” system1 and is intended for use by experienced C/C++ programmers. It is not a newbie learning tool.

   

3. ***WiringPi*** is developed directly on a Raspberry Pi running 32-bit Raspbian.**I do not support any other platform, cross compiling or operating systems.** 

[Raspberry Pi | Wiring | Download & Install | Wiring Pi](http://wiringpi.com/download-and-install/)

> note:如果官网地址打不开，直接下载github中的下载包

###### install

~~~bash
wget https://github.com/WiringPi/WiringPi/archive/refs/tags/2.61-1.tar.gz

tar zxvf 2.61-1.tar.gz 

cd WiringPi-2.61-1/

./build

OK
~~~



###### test

~~~shell
gpio -v
~~~

![img](https://img2023.cnblogs.com/blog/1908118/202403/1908118-20240314123334076-1247222859.png)
~~~shell
gpio readall
~~~

![image-20220504133213020](https://img2023.cnblogs.com/blog/1908118/202403/1908118-20240314123429056-1957084155.png)

## wiringpi API

参考博客

[树莓派wiringPi库详解 - lulipro - 博客园 (cnblogs.com)](https://www.cnblogs.com/lulipro/p/5992172.html)

[树莓派WiringPi常用函数中文手册-Arduino中文社区 - Powered by Discuz!](https://www.arduino.cn/thread-21348-1-1.html)

[树莓派 wiringPi 库_~莘莘的博客-程序员宝宝_wiringpi - 程序员宝宝 (cxybb.com)](https://www.cxybb.com/article/lcx1837/108121837)

~~~shell
//Core wiringPi functions

extern struct wiringPiNodeStruct *wiringPiFindNode (int pin) ;
extern struct wiringPiNodeStruct *wiringPiNewNode  (int pinBase, int numPins) ;

extern void wiringPiVersion	(int *major, int *minor) ;
extern int  wiringPiSetup       (void) ;
extern int  wiringPiSetupSys    (void) ;
extern int  wiringPiSetupGpio   (void) ;
extern int  wiringPiSetupPhys   (void) ;

extern          void pinModeAlt          (int pin, int mode) ;
extern          void pinMode             (int pin, int mode) ;
extern          void pullUpDnControl     (int pin, int pud) ;
extern          int  digitalRead         (int pin) ;
extern          void digitalWrite        (int pin, int value) ;
extern unsigned int  digitalRead8        (int pin) ;
extern          void digitalWrite8       (int pin, int value) ;
extern          void pwmWrite            (int pin, int value) ;
extern          int  analogRead          (int pin) ;
extern          void analogWrite         (int pin, int value) ;


~~~

