+++
title= "Docker安装"
description= "docker"
date= 2022-05-14T01:12:54+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    " docker"
]

+++

# docker安装

[Docker 官方文档 | Docker Documentation](https://docs.docker.com/)

## 1. window安装

需要安装wsl2

********

## 2.  linux

### 2.1. ubuntu安装

1.系统需求

- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Impish 21.10
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)

Docker Engine is supported on `x86_64` (or `amd64`), `armhf`, `arm64`, and `s390x` architectures.

2.卸载旧版本

~~~shell
sudo apt-get remove docker docker-engine docker.io containerd runc
~~~

3.使用仓库安装(也可以使用二进制包安装)

~~~shell
sudo apt-get update

 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

~~~

4.添加 GPG key

~~~
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
~~~

5.添加docker源

~~~shell
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
~~~

6. 安装 Docker Engine

~~~shell
 #安装最新docker
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
 
 #安装指定版本
 apt-cache madison docker-ce
 udo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
~~~

7.测试

~~~shell
docker version

sudo docker run hello-world
~~~

8.卸载 Docker Engine🔗

~~~shell

//1.Uninstall the Docker Engine, CLI, Containerd, and Docker Compose packages:
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-compose-plugin


//2.Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

~~~

## 3.docker加速

[轻量应用服务器 安装 Docker 并配置镜像加速源-最佳实践-文档中心-腾讯云-腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/1207/45596)

