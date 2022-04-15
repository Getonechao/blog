+++
title= "《C++ Effective》-读书笔记5"
description= "55具体做法-5(6继承与面向对象的设计)"
date= 2021-04-15T12:51:11+08:00
author= "chao"
draft= true
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    "cpp"
]

+++

# 继承与面向对象设计

## 条款32：确定你的public继承塑模出“is-a”（是一种）关系

## 结论

“**public继承关系**”意味is-a。**适用于base classes身上的每一件事情一定也适用于derived classes身上，每一个derived classes对象也是一个base class对象**



### 存在于classes之间的三种关系：

1. is-a （是一个）A是B
2. has-a（有一个，条款38）A有B
3. is-implemented-in-term-of(根据某物实现出，条款39)



## 条款33：避免遮掩继承而来的名称

### 结论

1. derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此。
2. 为了让被遮掩的名称再见天日，可以使用**using声明式**或**转交函数**。





## 条款34：区分接口继承和实现继承

## 结论

1. 接口继承和实现继承不同。在public继承之下，derived classes总是继承base class的接口。

2. pure vitrual（纯虚函数）只具体指定接口继承。
3. impure vitrual(非纯函数)具体指定指定接口继承和缺省实现继承（可以覆写）。
4. non-virtual（成员函数）具体指定接口继承以及强制性实现继承。



## 条款35：考虑virtual函数以外的其他选择

## 结论

1. virtual函数的替代方案包括NVI手法及Strategy（策略）设计模式的多种形式。NVI手法自身是一种特殊形式的Template Method设计模式。
2. 将机能从成员函数移到class外部函数，带来的一个缺点是，非成员函数无法访问class的non-public成员。
3. trl::function对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式兼容”的所有可调用物



## 条款36：绝不重新定义继承而来的non-virtual函数

### 原因

违反"is-a"的继承体系



## 条款37： 绝不重新定义继承而来的缺省参数值

### 结论

绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而virtual函数---你唯一应该覆写的东西----却是动态绑定。



### 原因

你只能继承两种函数：virtual 和 non-virtual函数。然而重新定义一个继承而来的non-virtual函数永远是错误（条款36）

**virtual函数系动态绑定，而缺省参数值却是静态绑定**。



## 条款38：通过复合塑模出“has-a”或“根据某物实现出”

## 结论

1. **复合关系**的意义和**public继承关系**完全不同
2. 在应用域，在复合意味has-a（有一个）。在实现域，复合意味is-implemented-in-terms-of(根据某物实现出)。



### 内容

继承关系 复合关系

在程序员之间，复合这个词有很多同义词，譬如分层、内含、聚合、内嵌。

###### 应用域

相当于塑造的世界中的某些事物，例如人、汽车

###### 实现域

实现细节上的人工制品，例如缓冲区、互斥器、查找树



## 条款39：明智而审慎地使用private继承

### 结论

1. private继承关系意味着is-implemented-in-terms-of(根据某物实现出)。它通常比复合的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计是合理的。
2. 和复合不同，private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。



### 原因

C++将public继承视为is-a关系

​	将private继承视为is-implemented-in-terms-of(根据某物实现出)





#### 注意：private继承纯粹只是一种实现技术（private base class的每样东西在你的class内都是private）

如非必要（必要条件，见P188），在复合与private继承中，尽量选择复合。





## 条款40：明智而审慎地使用多重继承

### 结论

1. 多重继承比单一继承复杂。它可能导致新的歧义性，以及对virtual继承的需要。
2. virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virtual base classes不会带任何数据，将是最具实用价值的情况。
3. 多重继承的确有正当用途。其中一个情景涉及“public继承某个interface class”和“private继承某个协助实现的class”两相组合。



