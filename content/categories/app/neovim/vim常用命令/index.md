+++
title= "Vim常用命令"
description= "文章简介"
date= 2022-09-12T04:17:39+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    " vim"
]

+++

# vim常用指令

![img](images/92c2e5fe9a07297bf14f418e1e420a06.gif)



![](images/f72af86e2ca97e1566be24c38b95ee57.png)



## 窗口命令

新建横向窗口

~~~
:sp
~~~

新建纵向窗口

~~~
:vs
~~~

切换窗口

~~~
ctrl+w w 
ctrl+w h j k l
ctrl+w t(左上) b(右下)
~~~

窗口移动

~~~

ctrl+w H 最左端
ctrl+w L 最右端
ctrl+w J 最低端
ctrl+w K 最高端
ctrl+w r 顺时针
ctrl+w R 逆时针
ctrl+w x 左右上下对应位置的窗口 对调
~~~



调整窗口高度

~~~
:resize 20 | +20 | -20
:vert resize 20 | +20 | -20
~~~



关闭窗口

~~~
ctrl+w  q(quit), c(close), o(other)
~~~

目录浏览

~~~
:He!(上分屏)
:He(下分屏)
:Ve!
:Ve

~~~

## Tab标签页

~~~
:Te (tab标签)

:tabnew 增加一个标签

:tabc       关闭当前的tab

:tabo       关闭所有其他的tab

:tabs       查看所有打开的tab :tabp 或gT 前一个

:tabn 或gt  后一个 新建标签页

:tabe 在新标签页中打开指定的文件。

:tabnew 在新标签页中编辑新的文件。

:tab split 在新标签页中，打开当前缓冲区中的文件。

:tabf 允许你在当前目录搜索文件，并在新标签页中打开


Vim默认最多只能打开10个标签页。你可以用set tabpagemax=15改变这个限制。
~~~







## 代码自动补全指令

当前项

~~~
ctrl+y  
~~~

上一项

~~~
ctrl+p
~~~

下一项

~~~
ctrl+n
~~~

保持当前文字输入，并退出补全

~~~
ctrl+e
~~~

## 终端命令

终端窗口

~~~
:term bash

:term (cmd)
~~~

终端命令

~~~
:!commond
eg:
:!gcc -v
~~~

