+++
title= "《C++ Effective》-读书笔记3"
description= "55具体做法-3(4接口的设计与声明)"
date= 2021-04-15T12:47:11+08:00
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

# 接口的设计与声明

接口设计的准则：
				让接口容易被正确使用，不容易被误用。



什么是接口？

​			内部实现细节封装起来，外部用户用过预留的接口可以使用接口的功能而不需要知晓内部具体细节。

​			C++中，通过类实现面向对象的编程，而在**基类中只给出纯虚函数的声明，然后在派生类中实现纯虚函数的具体定义的方式**实现接口，不同派生类实现接口的方式也不尽相同，从而实现多态。



## 条款18：	让接口容易被正确使用，不容易被误用。

	### 结论

1. 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
2. “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
3. “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
4. trl::shared_ptr支持定制删除器。这可防范DLL问题，可被用来自动解除互斥锁（条款14）。

### 原因

首先必须考虑客户可能做出什么错误。

~~~C++
class Date{
    public：
        Date（int month，int day，int year）;
    .....
}
~~~

如果年月日的顺序错误呢？

例如

Date d（30,3,1995）错误

亦或者按错

Date d（2,30,1995）

### 解决方法

导入简单的外覆类型来区别天数、月份、年份，然后于Date构造函数中使用这些类型：

~~~C++
struct Day
{
    explicit Day(int d):val(d){}
    int val;
};


struct Month
{
    explicit Month(int m):val(m){}
    int val;
};

struct Year
{
    explicit Year(int y):val(y){}
    int val;
};


class Date{
    Public:
	Date（const Month& m,const Day& d,const Year& y）;//包装了一下
    ...
}

Date d（30,3,1995）;		//错误！
Date d（Day（30），Month（3），Year（1995））；//错误
Date d(Month(3),Day(30),Year(1995))//OK,类型正确
~~~

## 条款19：设计class犹如type

### 结论

class的设计就是type的设计。在定义一个新type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。

### 如何设计高效的class呢？

1. 新type的对象应该如何被创建和销毁？



2. 对象的初始化和对象的赋值该有什么样的差别？



3. 新type的对象如果被passed by value(以值传递)，意味着什么？





## 条款20：宁以pass-by-reference-to-const替换pass-by-value

## 结论

1. 尽量以Pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可避免切割问题。
2. 以上规则并不适用内置类型、STL的迭代器和函数对象。对它们而言，pass-by-value往往比较合适。



## 条款21：必须返回对象时，别妄想返回其reference







## 条款22：将成员变量声明为private

从封装角度来看：其实只有两种访问权限：private（提供封装）和其他（不提供封装，即public、protected）

## 条款23：宁以non-member、non-friend替换member函数

### 结论

宁以non-member、non-friend替换member函数。这样做可以增加封装性、包裹弹性和机能扩充性

### 原因

一个例子

想象有一个class用来表示网页浏览器。class可能提供的许多函数中，有一些用来清除下载元素高速缓存区、清除访问过的URLs的历史记录、以及移除系统中的所有cookies：

~~~C++
class WebBrowser{
    public：
       ...
    void clearCache(); //清除高速缓存
    void clearHistory();//清除历史记录
    void removeCookies();//清除cookies
    ...
}
~~~

许多用户想一个函数执行所有动作，因此webBrowser也提供这样一个函数：

~~~C++
方法一：
class WebBrowser
{
    public:
    ...
        void clearEverything();//调用clearCache（），clearHistory（）和removeCookies（）
}


方法一的行为可以由non-member函数调用适用的member函数而提供出来（方法二）：

方法二：
    void clearBrowser（WebBrowser& wb）
{
     wb.clearCache(); //清除高速缓存
     wb.clearHistory();//清除历史记录
    wb.removeCookies();//清除cookies
}
~~~



方法一与方法二，哪一个更好一些呢？

从封装性考虑，为保护数据的封装性（private），应该选择方法二。

**封装的目的：愈多的东西被封装，我们改变那些东西的能力也就越大。愈多函数可访问它，数据的封装性就愈低。**



**能够访问private成员变量的函数**只有class的**member函数+friend函数**而已。

因为成员函数（member）可以访问无限制的访问private的数据，而non-member、non-friend函数并不能增加“能访问class内之private成分”的数据。



### 铭记：如果要你在一个member函数（它不只是可以访问class内的private数据，也可以取用private函数、enums、typedefs等等）和一个non-member，non-friend函数（它无法访问上述的任何东西）之间做抉择，而且两者提供相同机能，那么，导致较大封装性的是non-member non-friend函数，因此选择后者。



注意：non-member函数也可以是另一个class的member

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数

### 结论

如果你需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个non-member

















