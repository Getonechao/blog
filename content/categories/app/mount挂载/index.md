+++
title= "Mount：网络文件夹挂载"
description= "文章简介"
date= 2022-05-09T16:28:55+08:00
author= "somebody"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    "linux"," tool"
]

+++

# Mount挂载

## mount网络挂载



##  1 win10上新建一个文件夹



![image-20220510101839548](images/image-20220510101839548.png)

修改属性，设置共享



<img src="images/image-20220510102156548.png" alt="image-20220510102156548" style="zoom:50%;" />

如果不确定win10的用户名和密码，就新建一个本地用户

<img src="images/image-20220510102225088.png" alt="image-20220510102225088" style="zoom:50%;" />

<img src="images/image-20220510102257001.png" alt="image-20220510102257001" style="zoom:50%;" />

<img src="images/image-20220510102456203.png" alt="image-20220510102456203" style="zoom:50%;" />

## 2 linux挂载命令

查看当前linux用户id

~~~shell
id 
~~~

![image-20220510103204731](images/image-20220510103204731.png)

挂载

~~~shell
sudo mount -t cifs -o username=chao2,password=88888888zc! -o rw,uid=1000,gid=1000  //192.168.209.1/shrae_file /home/chao/Desktop/mountpoint
~~~

## 3 other

windows的cifs的端口是445

~~~shell
端口映射

树莓派(车体控制)
sudo iptables -t nat -I PREROUTING -p tcp --dport 445 -j DNAT --to-destination 192.168.8.50:445
sudo iptables -t nat -I POSTROUTING -p tcp -d 192.168.8.50/24   --dport 445 -j SNAT --to-source 192.168.8.101

ros(导航)
sudo mount -t cifs -o username=chao2,password=88888888zc! -o rw,uid=1000,gid=1000  //192.168.20.3/shrae_file /home/ubuntu/mount
~~~

