+++
title= "《C++ Effective》-读书笔记3"
description= "55具体做法-4(5实现)"
date= 2021-04-15T12:50:11+08:00
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

# 实现

大多情况下，适当提出你的class（和class templates）定义以及functions(和 function templates)声明，是花费最多心力的两件事。

实现大多直截了当，但实现仍然有一些东西要小心。





## 条款26：尽可能延后变量定义式的出现时间

### 结论

尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序的效率。

### 原因

只要你定义了一个变量而其类型带有一个构造函数或析构函数，

那么**当程序的控制流到达这个变量定义式时，你便得承受构造成本；**

**当这个变量离开其作用域时，你便要承受析构成本。**

即使这个变量最终并未被使用，仍需消耗这些成本。

~~~C++
std::string encrytPassword(const std::string&　password)
{
    using namespace std;
    string encrypted;
    if(password.length()<MinimumPasswordLength){
        throw logic_error("Password is too short");
    }
    ...	//必要动作，将一个加密后的密码置入变量encrypted内
   return encrypted;
}
~~~

如果密码长度不够，丢出异常，那么string encrypted被构造了，也被析构了，但没有被使用。

### P116重要，关于循环体内的定义







## 条款27：尽量少做转型动作



## 结论

1. 如果可以，尽量避免转型，特别是注重效率的代码中避免dynamic_casts。如果有个设计需要转型设计，试着发展无需转型的替代设计
2. 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需要转型放进它们自己的代码内。
3. 宁可使用C++style（新式）转型，不要使用旧式转型。





## 条款28：避免返回handles指向对象内部成分（即它的成员变量）

## 结论

避免返回handles（包括references、指针、迭代器）指向对象内部。遵守这个条约可增加封装性，帮助cosnt成员函数的行为更像const，并将发生内存对象被销毁的可能性降到最低。



### 原因

假设你的程序涉及矩形，每一个矩形由其左上角和右下角表示。

~~~C++

class Point{	//这个class用来表述“点”
    public：
        point（int x,int y）;
    ...
        void setX（int newVal）;
    	  void setY(int newval);
    ...
}

struct RectData{	//这些点数据用来表现一个矩形
    Point ulhc;	//ulhc="右下角"
    Point lrhc;//lrhc="右上角"
};

class Rectangle{
    ....
        private:
    std::trl::shared_ptr<RectData> pData;
};
~~~

by reference 方式传递用户自定义类型往往比by value方式更高效

~~~C++
class Rectangle{
    public：
        ....
        Point& upperLeft() const { return pData->ulhc;}
    	  Point& upperRight() const { return pData->lrhc;}
}
~~~



**const成员函数：若将成员成员函数声明为const，则该函数不允许修改类的数据成员，但若数据成员是指针，则还是可以修改指针指向的内容**



upperLeft和upperRight被声明为const成员函数，是为了提供客户一个得知Rectangle相关坐标点的方法，而不是让客户修改Rectangle（客户是可以通过by reference修改内部数据）。

### 解决方法

它们的返回类型加上const

~~~C++
class Rectangle
{
    public：
        ...
        const Point& upperLeft() const{ return pData->ulhc;}
    	const Point& lowerRight() const {return pData->lrhc;}
    ...
}
~~~



## 条款29：为“异常安全”而努力是值得的

## 结论

1. 异常安全函数即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型
2. “强烈保证”往往能够以copy-and-swap实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。
3. 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

## "COPY-AND-SWAP"技术

按照C++ primer的理解，赋值运算符（A=B）应该实现两个方面的工作：

1.析构函数（**删除A原有的内容**）

2.拷贝构造函数（**将B的值赋值给A）**



***



ClassA& ClassA::operator= (....): 首先把"="右边的值复制到"="左边, 然后析构"="左边的值;

为了保证可以自赋值(self-assignment), 需要使用***\*临时变量\****存储, 再删除对象;

~~~C++

class A {
private:
    int *b;
    int a;
public:
    A():a(0),b(nullptr){};
    A(const A&rhs):a(rhs.a),b(rhs.b==nullptr?nullptr:new int(*rhs.b)){};
    ~A(){
        delete b;
        b = nullptr;
    };
};


A& operator=(const A& rhs) {
    if(this!=&rhs) { // 防止自赋值
        delete b;
        this->b = new int(*rhs.b);// 可能失败
        this->a = rhs.a;
    }
    return *this; // 返回this对象的引用
}
~~~

可以看到我们的代码几乎是对**拷贝构造函数和析构函数的完全复制**，此外，上述代码虽然完成了自赋值的验证，但并未保障异常安全。**一旦new失败，原this对象的b已经被删除，因此会引发异常**。





####     异常不安全主要在于，b对应的对象可能在异常到来之前被删除。因此我们首先保存该对象的副本，从而保证了异常安全特性，无论new是否成功，this对象中的b指针都会指向已知对象

~~~C++
A& operator=(const A& rhs) {
    auto orign = this->b;		//使b指针所指向的内存有orgin指针指向
    this->b = new int(*rhs.b);
    delete orign;
    this->a = rhs.a;
    return *this;
}
~~~

#### copy and swap





为了使用copy-swap，我们需要三件事：

- 一个有效的拷贝构造函数
- 一个有效的析构函数
- 一个自定义的交换函数，不能用std::swap,因为该函数实现中调用了拷贝构造和复制函数，且交换函数不抛异常







该技术的核心就是**不再使用引用作为赋值运算符参数**，形参将直接是对象，这样的写法将会使编译器自动调用拷贝构造函数，由于拷贝构造函数的调用，异常安全将在进入函数体之前被避免（若拷贝失败则什么都不会发生）。经过swap后的对象在离开函数体后会自动销毁，因此也就自动调用了析构函数，具体写法如下：
————————————————
版权声明：本文为CSDN博主「feifeiiong」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/feifeiiong/article/details/77866579

~~~C++

void swap(A& rhs) {
    using std::swap;
    swap(this->a,rhs.a);
    swap(this->b,rhs.b);
}

A& operator=(A rhs) {
    swap(rhs);
    return *this;
}
~~~

网络资源：

1. https://blog.csdn.net/hiwubihe/article/details/116667884?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162631218716780264010481%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162631218716780264010481&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-10-116667884.first_rank_v2_pc_rank_v29&utm_term=copy+and+swap&spm=1018.2226.3001.4187

## 条款30：透彻了解inlining的里里外外

### 结论

1. 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更为容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
2. 不要只因为function templates出现在头文件，就将它们声明为inline。

### 原因

**一开始先不要将任何函数声明为inline**，或者至少将inline施行范围局限在那些“一定成为inline（条款46）”或“十分平淡无奇”的函数身上，可以使程序达到优化。

（28法则）平均而言一个程序往往将80%的执行时间花费在20%的代码上头，**作为软件开发者，我们的目标是找出这可以有效增进程序整体效率的20%的代码**



inline缺点：

以代码膨胀为代价 空间换时间

inline对于编译器而言，只是建议。

建议：

1. 开栈的开销 > 执行的开销 建议设为inline
2. 开栈的开销 < 执行的开销 不建议

### 为什么使用inline关键字？

###### 1.1为了解决一些频繁调用的小函数大量消耗栈空间（栈内存）。

在预编译的时候，编译器将程序中出现的内联函数的调用表达式的地方直接插入用内联函数的代码。





## 条款31：将文件间的编译依存关系降到最低

### 结论

1. 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classes和interface classes
2. 程序库头文件应该以“完全且仅有声明式”的形式存在。这种做法不论是否涉及templates都适合。

### 方法

pimpl idiom技术

![img](index.assets/20170728172522266)

https://blog.csdn.net/qq_33775402/article/details/76274678?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162632039416780261938712%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162632039416780261938712&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-4-76274678.first_rank_v2_pc_rank_v29&utm_term=%E9%99%8D%E4%BD%8E%E6%96%87%E4%BB%B6%E9%97%B4%E7%9A%84%E7%BC%96%E8%AF%91%E4%BE%9D%E5%AD%98%E5%85%B3%E7%B3%BB&spm=1018.2226.3001.4187