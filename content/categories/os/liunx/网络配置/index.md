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
ip addr add 192.168.8.30 dev eth0
~~~

### /etc/netplan/*配置文件
服务器中是50-cloud-init.yaml,桌面版01-network-manager-all.yaml

信息格式如下：
~~~
network:
  version: 2
  renderer: NetworkManager
    ethernets:
        eth0:
            addresses: [192.168.8.123/24]
            gateway4: 192.168.8.1
            dhcp4: true
            nameservers:
                addresses: [8.8.8.8,114.114.114.114]
            optional: true
    wifis:
        wlan0:
            access-points:
                chao:
                    password: '88888888'
            dhcp4: true
            optional: true
~~~

