+++
title= "网络配置"
description= "对linux(基于ubuntu18.04)网络的一些配置"
date= 2022-04-01T13:11:10+08:00
author= "张超"
draft= false
image= "" 
categories= [
    "os"
]

tags=  [
    "linux","net"
]
+++

# 网络配置

## ubuntu18.04配置IP

### 使用ip addr
~~~
只限于以太网配置(临时)
ip addr add 192.168.8.30/24 dev eth0
~~~

### /etc/netplan/*配置文件
服务器中是50-cloud-init.yaml,桌面版01-network-manager-all.yaml

信息格式如下：
~~~
network:
    ethernets:
        eth0:
            dhcp4: no
            addresses: [192.168.30.201/24]
            optional: true
        eth1:
            dhcp4: no
            addresses: [192.168.30.202/24]
            optional: true
    version: 2
    wifis:
        wlan0:
            access-points:
                chao:
                    password: '88888888'
            dhcp4: true
            optional: true
~~~