+++
title= "《C++ Effective》-读书笔记2"
description= "55具体做法-2（3资源管理）"
date= 2022-04-15T12:47:11+08:00
author= ""
draft= true
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    "C++"
]

+++

# 资源管理

资源：**一旦用了它，将来必须还给系统**。

譬如：动态分配内存、文件描述器、互斥锁、图形界面中的字型和笔刷、数据库连接、网络sockets。

## 条款13：以对象管理资源

创建一个对象管理资源，**资源管理类**

### 结论

​	1.为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。

​	2.两个常被使用的RAII classes分别trl::share_ptr和auto_ptr。前者通常是较佳选择，因为其copy行为直观。

## 条款14：在资源管理中，小心copying行为

copying行为也应该参考

[浅拷贝&&深拷贝](../100语法问题/003浅拷贝&&深拷贝.md)

### 结论

1.复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。

2.普遍而常见的RAII class copying行为是：禁止copying、施行引用计数法。

### 原因

资源管理类对象被复制，是一件不合理的行为，因为例如“互斥锁”、“动态内存分配”、等是具有唯一性的，两个对象的指针同时指向同一块内存，那么对象销毁时，内存释放，就必然一个指针称为悬空指针。

### 解决方法

因此：有两种做法：

1. 禁止复制--------详细地见，条款6，将copying操作声明为private。

   

2. 对底层资源祭出“引用计数法”。

   有时候我们希望保存资源，直到它最后一个使用者（某对象）被销毁。



~~~C++
class Lock{
    public：
        explict Lock（Mutex* pm）:mutexPtr(pm,unlock)//以某个Mutex初始化share_ptr,并以unlock函数为删除器
        {
            lock（mutePtr.get()）;//条款15
        }
    private：
        std::trl::shared_ptr<Mutex> mutexPtr;//使用share_ptr
}
~~~

shared_ptr(智能指针)的妙用。

本例的Lock Class不再声明析构函数，因为没用必要。

mutex的析构函数会在互斥器的引用次数为0时自动调用trl：share_ptr的删除器（本例为unlock）。

## 条款15：在资源管理类中提供对原始资源的访问

### 结论：

1. APIs往往要求访问原始资源，所以每一个RAII class应该提供一个“取得其所管理之资源”的方法。
2. 对原始资源的访问可能经由显式转换或隐式转换。一般而言，显式转换比较安全，但隐式转换对客户比较方便。



### 做法

trl::share_ptr和auto_ptr都提供一个get成员函数，用来执行显式转换，也就是它会返回智能指针内部的原始指针（的复件）



## 条款16：成对使用new和delete时要采取相同形式

### 结论

如果你在new表达式中使用[],必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[],一定不要在相应的delete表达式中使用[]



## 条款17：以独立语句将newed对象置入智能指针

### 结论

以独立语句将newed对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。

### 原因

假设我们有个函数用来揭示处理程序的优先权，另一个函数用来在某动态分配所得的widget上进行某些带有优先权的处理：

~~~C++
int priority();
void processWidget(std::trl::shared_ptr<Widget> pw, int priority);
~~~

调用processWidget

~~~C++
processWidget(new Widget, priority());
~~~

不能通过编译，因为trl::share_ptr构造函数需要一个原始指针，但该构造函数是个explicit构造函数，无法进行隐式转换，需要将“new Widget”的原始指针转换为processWidget所要求的Trl：share_ptr。

但

这样，就可以通过编译。

~~~C++
processWidget(std::trl::share_ptr<Widget>(new Widget), priority());
~~~

强制转换

缺陷是：可能泄漏资源

假设执行顺序：

1. 执行“new Widget”
2. 调用priority
3. 调用trl::shared_ptr构造函数

对priority的调用可以排在第一、第二、第三执行，不一定，不可知。

如果在第二位执行，如果对priority的调用导致异常呢？那么“new Widget”返回的指针将会遗失，因第三步执行不了，而第三步则是我们期盼用来防卫资源泄漏的武器，RAII。

### 解决方法

使用分离语句

（1）创建widget

（2）将它置入一个智能指针，然后再把那个智能指针传给processWidget

~~~C++
std::trl::shared_ptr<Widget> pw(new Widget);//在单独语句内以智能指针存储newed所得的对象。



processWidget(pw,priority());//不会造成内存泄漏
~~~

