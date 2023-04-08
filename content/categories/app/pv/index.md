+++
title= "pv：Pipe Viewer 通过管道显示数据处理进度的信息"
description= "文章简介"
date= 2023-04-08T22:44:43+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["app"]

tags=  ["linux "," "]

+++

# 一、简介

Pipe Viewer 的简称，意思是通过管道显示数据处理进度的信息。特别适合某些场景比如拷贝文件，不显示进度，可以用PV显示

~~~
pv(选项)(参数)

-p, --progress           显示进度条				【默认使用】
-t, --timer              显示已用时间 		    【默认使用】
-e, --eta                显示预计到达时间 (完成)	 【默认使用】
-I, --fineta             显示绝对估计到达时间
                         (完成)
-r, --rate               显示数据传输速率计数器	【默认使用】
-a, --average-rate       显示数据传输平均速率计数器
-b, --bytes              显示传输的字节数		  【默认使用】
-T, --buffer-percent     显示正在使用的传输缓冲区百分比
-A, --last-written NUM   显示上次写入的字节数
-F, --format FORMAT      将输出格式设置为FORMAT
-n, --numeric            输出百分比
-q, --quiet              不输出任何信息

-W, --wait               在传输第一个字节之前不显示任何内容
-D, --delay-start SEC    在SEC秒过去之前不显示任何内容
-s, --size SIZE          将估算的数据大小设置为SIZE字节
-l, --line-mode          计算行数而不是字节数 
-0, --null               行以零结尾
-i, --interval SEC       每SEC秒更新一次
-w, --width WIDTH        假设终端的宽度为WIDTH个字符 
-H, --height HEIGHT      假设终端高度为HEIGHT行
-N, --name NAME          在可视信息前面加上名称
-f, --force              将标准错误输出到终端
-c, --cursor             使用光标定位转义序列

-L, --rate-limit RATE    将传输限制为每秒RATE字节
-B, --buffer-size BYTES  使用BYTES的缓冲区大小
-C, --no-splice          从不使用splice()，始终使用读/写
-E, --skip-errors        跳过输入中的读取错误
-S, --stop-at-size       传输--size字节后停止
-R, --remote PID         更新过程PID的设置

-P, --pidfile FILE       将进程ID保存在FILE中 

-d, --watchfd PID[:FD]   监视进程PID,打开的文件FD

-h, --help               显示帮助
-V, --version            显示版本信息
~~~

# 二、使用方法

## 2.1 复制文件

~~~
如果没有指定选项，默认使用 -p, -t, -e, -r 和 -b 选项
pv getiot.db > getiot.db.bak
~~~



