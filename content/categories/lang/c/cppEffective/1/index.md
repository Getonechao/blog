+++
title= "《C++ Effective》-读书笔记1"
description= "55具体做法-1（1-2构造、析构、赋值运算）"
date= 2022-04-15T12:46:11+08:00
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

##  条款05： C++默认编写并调用那些函数

默认创建

```c++
class Base
{
    public：
    defualt构造函数
    copy构造函数
    copy assignment操作符
    析构函数
}

```



## 条款06：若不想使用编译器自动生成的函数，就该明确拒绝（即不可被调用）

作法：

​	将默认创建的函数声明为私有的（——private——），为防止member函数和friend函数的内部调用，**将成员函数声明为private而且故意不实现它们**

```C++
class HomeForSale
{
    public:
    
    private:
    HomeForSale(const HomeForSale&);	//只有声明，不实现它们
    HomeForSale& operator=(const HomeForSale&);
}
    

```

更为常用的作法：
	设计一个基类（base class） ，在这个基类内实现阻止copying等默认函数动作，然后继承它。



## 条款07：为多态基类声明virtual析构函数

### 用法:

1.如果class不含virtual函数，通常表示它并不意图被用做一个base class。

​	即：class的设计目的如果不是作为base class使用，或者不是为了具备多态性，就不应该virtual析构函数。

2.当class不企图被当做base class，令其析构函数为virtual往往是个馊主意

3.多态性质的base classes应该声明一个virtual析构函数，**如果class带有任何virtual函数，他就应该拥有一个virtual析构函数**

### 目的：

1.C++中基类采用virtual虚析构函数是为了防止内存泄漏。

2.假设基类中采用的是非虚析构函数，当删除基类指针指向的派生类对象时就不会触发动态绑定，因而只会调用基类的析构函数，而不会调用派生类的析构函数

### 可参考blog

1. https://blog.csdn.net/sinat_20265495/article/details/51775724?ops_request_misc=&request_id=&biz_id=102&utm_term=%E6%9E%90%E6%9E%84%E5%87%BD%E6%95%B0%20virtual&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-8-.pc_search_result_before_js&spm=1018.2226.3001.4187
2. https://blog.csdn.net/yhc166188/article/details/81587442?ops_request_misc=%7B%22request%5Fid%22%3A%22162622287916780261951132%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D

## 条款08：被让异常逃离析构函数

###　结论

１．析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。

２．如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。

### 原因

****

为什么析构函数不应吐出异常？

vector的使用

```c++
class widget{
    public:

    ~widget(){}	//假设吐出一个异常
}

void dosomething()
{
    std::vector<Widget> v;
    
}			//v在这里被自动销毁

```



假设v内含十个Widgets,而在析构第一个元素期间，有一个异常被抛出，其他九个widgets还是应该被销毁。

这会导致多个异常同时存在，程序会结束执行或导致不明确行为。（结论1）

****

若析构函数需要吐出异常，怎么办?

一个例子：
	数据库的连接，创建一个用来管理DBConnection资源的class，并在其析构函数中调用close（）。但close（）函数可能关闭失败，从而输出异常。

~~~c++
class DBConn
{
    public:
    
    ~DBConn()//确保数据库连接总是会被关闭
    {
        db.close();//可能会关闭失败，输出异常
    }
    private:
	DBConnection db;
}
~~~



如果close（）失败，那么异常会离开~DBConn（）析构函数，成为麻烦。应该阻止异常从析构函数中传播出去。

解决办法：
**1.如果close抛出异常就结束程序。通常通过调用abort完成：**

即：

~~~C++
DBConn::~DBConn()
{
    try{db.close();}
    catch(...){
        制作运转记录，记录对close（）的调用失败（即日志库）;
        std::abort();
    }
}
~~~



**2.吞下因调用close而发生的异常（即对发生的异常，catch中不做处理）：**

~~~C++
DBConn::~DBConn()
{
   try{db.close();}
    catch(...){
        制作运转记录，记录对close（）的调用失败（即日志库）;
    }
}
~~~



### 具体做法

重新设计DBConn接口,给客户一个关闭close函数的机会

~~~C++
class DBConn{
    public：
       ...
       void close（）//供客户使用的新函数
    {
        db.close();
        closed=true;
    }
    ~DBConn()
    {
        if(!closed)
        {
            try{
                db.close();	//关闭连接（如果客户不主动关闭）
               }
   				catch(...){								//如果关闭动作失败
                    制作运转记录，记录对close的调用失败；//记录下来并结束程序 或 吞下异常
                }         
        }
    }
    private：
       DBConnectin db;
       bool closed; 
}
~~~

## 条款09：绝不在构造和析构函数过程中调用virtual函数

###　结论

在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class

### 原因

一个例子：

模拟股市交易如买进、卖出的订单等，每当创建一个交易对象，在审计日志中也需要创建一笔适当的记录。

~~~C++
基类
class Transaction{					//所有交易的base class
    public:
    Transaction();
    virtual void logTransaction() const = 0;//做出一份因类型不同而不同的日志记录
    
};


Transaction::Transaction()
{
    .....
   logTransaction();			//问题点
}

子类
class BuyTansaction:public Transaction{
    public:
    virtual void logTransaction() const; //log此记录
    ....
} 

class SellTansaction:public Transaction{
    public:
    virtual void logTransaction() const; //log此记录
    ....
} 


BuyTrasaction b;

~~~

当BuyTrasaction b后，首先Transaction构造函数一定会被先调用，然后BuyTransaction构造函数被调用。

Transaction::Transaction()中的logTransaction()函数执行的base class中的logTransaction。

### 解决方法



在class Transaction内将**logTransaction函数改为non-virtual**，然后要求**derived class构造函数传递必要信息给Transaction构造函数**，而后那个构造函数便可安全地调用non-virtual logTransaction。

即，通过构造函数的参数传递

1. 虚函数是从base class向下调用。
2. dervice class将必要的构造信息向上传递至base class构造函数。

~~~C++
注意形参
class Transaction
{
    public：
        explicit Transaction(const std::string& logInfo);
    	  void logTransaction(const std::string& logInfo) const;//如今是一个non-virtual函数
    ....
};

Transaction::Transacton(const std::string& logInfo)
{
    ...
        logTransaction(logInfo)
}

class BuyTransaction:public Transaction{
    public:
    BuyTransaction( parameters ): Transaction（createLogString（parameter））//将log信息传给base class构造函数
    {
        ...
    }
    ...
        
    private:
    static std::string createLogString( parameter );//注意这个函数返回string类型， 函数是static类型的，可以作为构造函数的参数使用
}
~~~



函数使用static类型，使之不可能意外指向“初期未成熟之BuyTransaction对象内部尚未初始化的成员变量”。