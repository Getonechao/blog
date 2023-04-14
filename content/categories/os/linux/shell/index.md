+++
title= "Linux：Shell脚本编程"
description= "shell编程"
date= 2023-04-14T12:49:07+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["os"]

tags=  [" Linux"," "]

+++

[README - 《阮一峰 Bash 脚本教程》 - 书栈网 · BookStack](https://www.bookstack.cn/read/bash-tutorial/README.md)

# 一、启动环境

~~~
当前bash环境
source main.sh

子bash环境
./main.sh
~~~

## 二、变量

## 2.1 将局部变量声明为全局变量

~~~
export a=1
~~~

## 2.2 位置变量

~~~
$0 $1 $2 $3  --- $10：($0 文件名本身)

$#：参数个数

$@：表示获取执行脚本传入的所有参数

$*：表示执行脚本传入参数的列表（不包括$0）

$?：表示脚本执行的状态，0表示正常，其他表示错误

$$：表示进程的id；Shell本身的PID（ProcessID，即脚本运行的当前 进程ID号）

$!：Shell最后运行的后台Process的PID(后台运行的最后一个进程的 进程ID号)
~~~

## 2.3 if条件判别式

形式

~~~
if [条件判别式]；then
	当条件判别式成立时
fi
~~~


~~~
&& 等同 AND
|| 等同 OR

if [条件判别式] AND [条件判别式];then

elif [条件判别式];then

else


fi

~~~

## 2.4 case.....esac判断

~~~
case $aNum in
          1)  echo 'You select 1'
          ;;
          2)  echo 'You select 2'
          ;;
          3)  echo 'You select 3'
          ;;
          4)  echo 'You select 4'
          ;;
          *)  echo 'You do not select a number between 1 to 4'
          ;;
esac
~~~

## 2.5 function功能

~~~
function fname(){
	程序段
}
~~~

function也是拥有内置变量的，它的内置变量与shell脚本很类似，函数名称代表示$0，而后续接的变量也是以$1、$2...来替换。

因为shell脚本的执行方式是由上往下，由左而右，因此在shell脚本当中的function的设置一定要在程序的最前面。

## 2.6 循环（loop）

模式一：满足什么条件开始循环

~~~
while [条件判别式]
do
	程序段落
done
~~~

模式二：

~~~
until [条件判别式]
do

done
~~~

## 2.7 for...do......done(固定循环)

模式一

~~~
for var in con1 con2 con3 ...
do
	程序段
done 
################################
注：除了$(seq 1 100)之外，seq是连续(sequence)的缩写之意，你也可以直接通过bash的内置机制来处理，可以使用{1.. 100}来替换$(seq 1 100)。也可以echo{a..g}
eg:
for var in $(seq 1 100)
do

done
~~~

模式二

~~~
for((初始值;限制值;赋值运算))
do
	程序段
done

###############################
eg:

for((i=1;i<=7;i=i+1))
do
	echo $i	
done
~~~

