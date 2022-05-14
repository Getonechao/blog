+++
title= "如何在wsl1中使用网络代理？"
description= "访问不了Google？git失败？"
date= 2022-03-23T20:09:16+08:00
author= "somebody"
draft= false
slug= "test-post"
image= "face.jpg" 
categories= [
    "app"
]

tags=  [
    " wsl1"
]

+++

# 如何在wsl1中使用网络代理？

## 代理工具

1. ### polipo

2. proxychain

## 前提条件

v2ray中开启允许局域网连接

![image-20220323201609014](index.assets/image-20220323201609014.png)

记住以下ip:port

![image-20220323201749350](index.assets/image-20220323201749350.png)

## 开始代理

### polipo

参考博客：[为 windows wsl 配置 socks5 代理 (github.com)](https://gist.github.com/moenn/2db47589724cf6c06ad9316ac57e2144)

步骤总结：

```
下载
sudo apt install polipo
打开配置文件
sudo nano /etc/polipo/config
写入
socksParentProxy = "localhost:10808"
socksProxyType = socks5
proxyPort = 8123

环境设置
nano ~/.bashrc
写入
export https_proxy=http://127.0.0.1:8123
export http_proxy=http://127.0.0.1:8123 
export all_proxy=socks5://127.0.0.1:8123

启动
sudo service polipo stop 
sudo service polipo start 

测试
curl www.google.com
```

### proxychain

参考博客：[linux下的全局代理工具proxychain | MonkeyWie's Blog](https://monkeywie.cn/2020/07/06/linux-global-proxy-tool-proxychain/)

