---
layout: post
title:  模板特例化
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
//设基本模板如下
template <class A, class B, class C>  
class X{};  
```

** 全部模板特例化 **
模板中所有参数全被指定为确定的类型

```
template<>
class X<int, float, double>{}
```

** 部分模板特例化分两种情况 **
1. 对部分模板参数进行特例化
```
template <class B, class C>
class X<int, B, C>{};
```
2. 使用具有某一特征的类型，对模板参数进行特例化
```
template<class A, class B, class C>
class X<A*,B*,C>{};
//当模板实参中前两个是指针时，编译器使用本特例化模板
```
混合这两种情况的例子如下
```
template<class A, class B>
class X<A*,B*,int>{};
```

** 总结：

template后<>的列表中要列出没有明确指明类型的 typename **

下面是一段示例代码：
```
#include <iostream>
#include <string.h>
using namespace std;

template<class T>
class A
{
public:
    T Max(T a, T b)
    {
        return (a>b)?(a):(b);
    }    
};

//class A针对类型char*的特例化
template<>
class A<char*>
{
public:
    char * Max(char *a, char *b)
    {
        return (strcmp(a,b))?(a):(b);
    }    
};

int main()
{
    A<int> obj;
    int x, y;
    cout << "Enter 1st integer:" << endl;
    cin >> x;
    cout << "Enter 2nd integer:" << endl;
    cin >> y;
    cout << "The bigger one is " << obj.Max(x,y) << endl;

    return 0;
}
```

下面是另一段sample代码：
```
#include <iostream>
#include <string.h>
using namespace std;

template<typename A, typename B>
class Base
{
public:
    Base(B x):value(x){}

    bool compare(A x, A y)
    {
        return (x>y)?(true):(false);
    }

    void print(void)
    {
        cout << "Basic template:" << value << endl;
    }

    B value;
};

template<typename A>
class Base<A*,int>
{
public:
    Base(int x):value(x){}
    bool compare(A* x, A* y)
    {
        return *x > *y ? true : false;
    }

    void print(void)
    {
        cout << "specialized template:" << value << endl;
    }

    int value;
};

int main()
{
    // basic template
    Base<int,float> a(2.1);
    a.print();    
    if (a.compare(39, 29))
        cout << "$1 > $2" << endl;
    else
        cout << "$1 <= $2" << endl;

    // specialized template
    Base<float*,int> b(7);
    float v1 = 1.2, v2=2.8;
    b.print();
    if (b.compare(&v1, &v2))
        cout << "$1 > $2" << endl;
    else
        cout << "$1 <= $2" << endl;

    return 0;
}
```
