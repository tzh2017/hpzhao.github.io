---
title: 如何禁止C++默认成员函数
date: 2016-03-02
tags: C++
categories: C++

---
# 前言 #

前几天在一次笔试过程中被问到C++如何设计禁止调用默认构造函数，当时简单的想法是直接将默认构造函数声明为private即可，这样的话对象的确不能直接调用。之后查阅了《Effective C++》之后得到了比较详尽的解释。  

# 了解C++的默认行为 #

当我们创建空类时，C++默认给我们生成了四种成员函数：  

1. 构造函数
2. 析构函数
3. 拷贝构造函数(*copy*)
4. 重载=的拷贝函数(*copy assignment*)  

因此，当你写下如下的代码：  

```C++
class Empty{};
```

那么编译器会自动生成：  

```C++
class Empty{
public:
    Empty(){...}                              //default构造函数
    Empty(const Empty& rhs){...}              //copy构造函数
    ~Empty(){...}                             //析构函数
    Empty& operator=(const Empty& rhs){...}   //copy assignment操作符
};

```  

至于copy构造函数和copy assignment操作符是不是有效取决于类的成员变量，例如：如果类成员有const变量或者引用，那么是不能重新赋值的。  

# 拒绝使用编译器自动生成函数 #

书中提到了一个实际的应用场景。在房子销售时，每一套房子都是独一无二的（地理位置，装修等等），那么显然我们不想让别人使用拷贝构造函数或者copy assignment操作符。但是如果我们不写，那么编译器会自动生成。如果我们写了就会可能让别人利用。那么该怎么办呢？  

+ 首先我们最自然的想法就是把这两个函数声明为私有，这样别人调用的时候可能会报错。我们的直觉是正确的。确实这样做可以行得通。于是我们如此写到：  

```C++
class HomeForSale{
public:
    ...
private:
    HomeForSale(const HomeForSale& hfs){...}
    HomeForSale& operator=(const HomeForSale& hfs){...}
};
``` 

+ 当然那么做并不是万事大吉了。因为member函数和友元函数仍然能调用私有成员函数。那么你可能又想到答案：我们无需定义成员函数，只需声明即可：  

```C++
class HomeForSale{
public:
    ...
private:
    HomeForSale(const HomeForSale&);
    HomeForSale& operator=(const HomeForSale&);
};
``` 

那么member成员函数和友元函数调用时，在编译阶段没问题，在链接阶段会报错。那么还有没有更好的方案能够让代码在编译阶段就能检测出错误呢？ 答案是肯定的。  

+ 
我们为此建立一个基类：  

```C++
class Uncopyable{
protected:
    Uncopyable(){}                            //允许derived对象的构造和解析
    ~Uncopyable(){}
private:
    Uncopyable(const Uncopyable&);            //但阻止copying
    Uncopyable& operator=(const Uncopyable&);
};
``` 

那么，为了阻止HomeForSale被拷贝，我们只需继承Uncopyable：  

```C++
class HomeForSale:private Uncopyable{
    ...
};
```  

+ 
C++11提出更为简洁的解决方案：  

```C++
class HomeForSale{
public:
    HomeForSale(const HomeForSale&) = delete;
    HomeForSale& operator=(const HomeForSale&) = delete;
};
```  
# 总结 #
+ 
为驳回编译器自动生成的成员函数，可将相应成员函数声明为private并且不予实现。或者使用像Uncopyable这样的基类。或者使用C++11的新特性。

+ 
在boost也有这样的基类:noncopyable。

