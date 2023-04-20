+++
title= "wsl：window下一款Linux环境"
description= "访问不了Google？git失败？"
date= 2022-03-23T20:09:16+08:00
author= "somebody"
draft= false
slug= "test-post"
image= "face.jpg" 
categories= [
    "os"
]

tags=  [
    " wsl"
]

+++

# 一、wsl与wsl2相互切换

## 1.开启windows相关功能

![image-20230412101652516](images/image-20230412101652516.png)

## 2.安装Windows升级软件

![image-20230412102121655](images/image-20230412102121655.png)

## 3.重启，然后“管理员终端”

~~~ shell
wsl --set-version <分发版名称 wsl -l -v 查看> 2或1
~~~





# 二、如何在wsl中使用网络代理？

wsl2需要在管理员终端，先开启windows防火墙权限

~~~
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
~~~

打开clash的端口

![](images/image-20230420163307553.png)

代理端口设置

~~~
nano ~/.bashrc
#加入，其中7890为clash的代理端口

hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:7890"
export http_proxy="http://${hostip}:7890"
export all_proxy="socks5://${hostip}:7890"
~~~

