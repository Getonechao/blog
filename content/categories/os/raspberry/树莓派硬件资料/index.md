+++
title= "Raspberry：硬件资料"
description= "40pin、wiringPi"
date= 2022-05-04T11:56:53+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["os"]

tags=  ["raspberry "]

+++

# 树莓派硬件资料

# 40pin



![img](images/rpi-pins-40-0.png)

- SDA.0、SDA.1：I2C数据传输口
- SCL.0、SCL.1：I2C的时钟信号
- GPIO.x（x = 0,1,2,3,4,5,6,7;21,22,23,24,25,26,27,28,29）:通用的输入输出，自己定义即可
- TXD\RXD: 串口
- MOSI：主输出  从输入(SPI)
- MISO：主输入  从输出(SPI)
- SCLK：SPI通信的时钟线(SPI)
- CE0、CE1：片选信号(SPI)

