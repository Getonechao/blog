+++
title= "一些部署博客的常用指令"
description= "command"
date= 2022-04-15T10:36:38+08:00
author= "somebody"
draft= false
image= "" 
math= true
categories= [
    "other"
]

tags=  [
    "blog","command"
]

+++



# command

## 1. 常用命令

### 1.1 新建文档

~~~shell
math类
hugo new categories/math/
--> hugo new categories/math/signalsAndSystems/

app类
hugo new categories/app/

os类
hugo new categories/os/

robat类
hugo new categories/robat/

lang类
hugo new categories/lang/

other类
hugo new categories/other/
~~~



### 1.2 上传命令

~~~
hugo -D && git add .&&git commit -m ""

git push github &&git push gitee &&cloudbase hosting deploy ./public  -e  blog-0g8860131649bb29
~~~

### 1.3 markdown技巧

- 打开调试，获取bilibili的aid

~~~shell

console.log(playerInfo.aid)

note：去除'\'
{\{< bilibili aid >}\}
~~~

- 跳转
~~~shell
[]({\{< ref "blog/neat.md" >}\})
~~~



## 2. 腾讯云部署

[静态网站托管 部署 Hugo - 最佳实践 - 文档中心 - 腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/1210/43389)