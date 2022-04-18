+++
title= "《信号与系统》读书笔记1--LTS"
description= "《信号与系统》奥本海默--线性时不变系统(LTS)"
date= 2022-04-17T15:40:32+08:00
author= "somebody"
draft= true
image= "" 
math= true
categories= [
    "math"
]

tags=  [
    "book ","project"
]

+++

# LTS

## 卷积的概念

> 连续系统:
>
> $$(f * g)(n)=\int_{-\infty}^{\infty} f(\tau) g(n-\tau) d \tau$$
>
> 离散系统:
>
> $$(f * g)(n)=\sum_{\tau=-\infty}^{\infty} f(\tau) g(n-\tau)$$

&emsp;&emsp;卷积这个名词的理解：**所谓两个函数的卷积，本质上就是先将一个函数翻转，然后进行滑动叠加。**在连续情况下，叠加指的是对两个函数的乘积求积分。在离散情况下就是加权求和，为简单起见就统一称为叠加。

{{< bilibili 713651125>}}

![](index.assets/OIP.JmUpfqJNtTPGMzVgTXGggwHaBZ)

 ## 离散时间线性时不变(LTS)系统：卷积和

- $x[n]=\sum_{k=-\infty}^{+\infty}x[k]\delta [n-k]$------$x[k]$是线性组合式中的权因子
- $y[n]=\sum_{k=-\infty}^{+\infty}x[k]h_{k} [n]$，令<font color="red">$h_{k}[n]$(即$h_{0}[n-k]$)为线性系统对移动单位脉冲$\delta[n-k]$的响应</font>
- LTS系统的单位脉冲响应可以完全刻画系统的特征。