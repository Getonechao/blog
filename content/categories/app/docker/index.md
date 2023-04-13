+++
title= "Docker：一个环境打包的容器"
description= "docker"
date= 2022-05-14T01:12:54+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["app”]

tags=  [" docker"]

+++

# 一、docker安装

[Docker 官方文档 | Docker Documentation](https://docs.docker.com/)



## 1.自动安装

~~~
wget http://fishros.com/install -O fishros && sudo bash fishros
~~~

********

## 2.手动安装

### 2.1. ubuntu安装

#### 1.系统需求

- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Impish 21.10
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)

Docker Engine is supported on `x86_64` (or `amd64`), `armhf`, `arm64`, and `s390x` architectures.

#### 2.卸载旧版本

~~~shell
sudo apt-get remove docker docker-engine docker.io containerd runc
~~~

#### 3.使用仓库安装(也可以使用二进制包安装)

~~~shell
sudo apt-get update

 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

~~~

#### 4.添加 GPG key

~~~
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
~~~

#### 5.添加docker源

~~~shell
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
~~~

6. #### 安装 Docker Engine

~~~shell
 #安装最新docker
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
 
 #安装指定版本
 apt-cache madison docker-ce
 udo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
~~~

#### 7.测试

~~~shell
docker version

sudo docker run hello-world
~~~

#### 8.卸载 Docker Engine🔗

~~~shell

//1.Uninstall the Docker Engine, CLI, Containerd, and Docker Compose packages:
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-compose-plugin


//2.Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

~~~

## 3.docker镜像元

[轻量应用服务器 安装 Docker 并配置镜像加速源-最佳实践-文档中心-腾讯云-腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/1207/45596)

# 二、docker常用命令

## 1.命令图

![查看源图像](images/v2-820aee2a33654099d87cdd2b7a1ce741_r.jpg)

## 2.docker run流程

![image-20220515022411757](images/image-20220515022411757.png)

## 3.docker 命令



### 3.1 帮助命令

~~~sh
    sudo docker version #显示docker版本信息

    sudo docker info #显示docker系统信息，包括镜像和容器的数量

    docker 命令 --help

~~~

### 3.2 镜像命令

1. ##### 查看所有镜像

   ~~~shell
   sudo docker images -a
   ~~~

2. ##### 搜索镜像

   ~~~shell
   //搜索stars数量大于500的镜像
   sudo docker search [镜像名] --filter=STARS=500
   ~~~

3. ##### 下载镜像

   ~~~shell
   sudo docker pull [镜像名]
   sudo docker pull [镜像名]:[版本名]
   ~~~

4. ##### 删除镜像

   ~~~shell
   sudo docker rmi -f [镜像ID]
   ~~~


### 3.3 容器命令

1. ##### 新建容器并启动

   ~~~shell
   sudo docker run [可选参数] [镜像名]
   
   #参数说明
   --name='name' 容器的名字
   -d			  后台交互运行
   -it  		  使用交互方式运行，进入容器查看内容
   -P			  指定容器的端口 -P 8080:8080
   -p			  随机指定端口
   
   
   //启动并进入容器
   sudo docker run -it [镜像名] /bin/bash
   
   //退出容器
   exit #直接容器停止并退出
   ctrl+p+q #容器不停止，退出
   
   ~~~

2. ##### 列出容器

   ~~~shell
   //列出所有正在运行中的容器
   sudo docker ps
   
   //列出所有正在运行中的容器+历史记录
   sudo docker ps -a
   
   //列出最近创建的前number个的容器
   sudo docker ps -a -n=number
   ~~~

3. ##### 删除容器

   ~~~shell
   //删除停止运行的容器
   sudo docker rm [容器id]
   
   //删除正在运行的容器
   sudo docker rm -f [容器id]
   
   //删除所有容器
   sudo docker rm -f $(docker ps -aq)
   
   ~~~

4. ##### 启动和停止容器

   ~~~shell
   //启动容器
   sudo docker start [容器ID]
   
   //重启容器
   sudo docker restart [容器ID]
   
   //停止当前正在运行的容器
   sudo docker stop [容器ID]
   
   //强制停止容器
   sudo docker kill [容器ID]
   
   ~~~

### 3.4 常用的其他命令



##### 日志信息

~~~shell
sudo docker logs  [容器ID ]
~~~

##### 容器进程信息

~~~shell
sudo docker top [容器ID ]
~~~

##### 容器元数据

~~~shell
sudo docker inspect [容器ID] 
~~~

##### 进入正在运行的容器

~~~shell
//进入容器后开启一个新的终端
sudo docker exec -it [容器ID]  /bin/bash
//进入容器执行的当前终端
sudo docker attach [容器ID] 
~~~

##### copy[容器文件copy到linux系统]

~~~shell
sudo docker cp [容器ID]:/home/chao/demo.txt /home/chao
~~~

