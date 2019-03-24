---
layout: post
title:  C++中explicit与隐式转换
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




普通构造函数可以被隐式调用，而explicit构造函数（显式构造函数）只能被显式调用

关于隐式转换，参考：【转】[这篇文章](https://www.cnblogs.com/QG-whz/p/4472566.html)

首先, C++中的explicit关键字只能用于修饰只有一个参数的类构造函数, 它的作用是表明该构造函数是显示的, 而非隐式的, 跟它相对应的另一个关键字是implicit, 意思是隐藏的,类构造函数默认情况下即声明为implicit(隐式)

《C++ Primer》中提到：
>可以用 单个形参来调用 的构造函数定义了从 形参类型 到 该类类型 的一个隐式转换

**这里应该注意的是， “可以用单个形参进行调用” 并不是指构造函数只能有一个形参，而是它可以有多个形参，但那些形参都是有默认实参的。 那么，什么是“隐式转换”呢？ 上面这句话也说了，是从 构造函数形参类型 到 该类类型 的一个编译器的自动转换。**
- 可以使用一个实参进行调用，不是指构造函数只能有一个形参
- 隐式类类型转换容易引起错误，除非你有明确理由使用隐式类类型转换，否则，将可以用一个实参进行调用的构造函数都声明为explicit
- explicit只能用于类内部构造函数的声明。它虽然能避免隐式类型转换带来的问题，但需要用户能够显式创建临时对象（对用户提出了要求）

```
#include <iostream>
#include <string.h>
using namespace std;
class Test1
{
public:
    Test1(int n) // 普通构造函数
    {num = n;}
private:
    int num;
};
class Test2
{
public:
    explicit Test2(int n) // （显式）构造函数
    {num = n;}
private:
    int num;
};
int main()
{
    Test1 t1 = 1;
    Test2 t2 = 2; //编译错误
    Test2 t3(3);
    return 0;
}
```
