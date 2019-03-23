---
layout: post
title:  C++纯虚函数使用example code 1
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <string.h>
using namespace std;

class base
{
public:
    virtual void func1(void) = 0;
    virtual void func2(void) = 0;
    void print(void)
    {
        func1();
        func2();
    }
};

class derived: private base
{
public:
    virtual void func1(void)
    {
        cout << "derived: func1()" << endl;
    }
    virtual void func2(void)
    {
        cout << "derived: func2()" << endl;
    }
    void print(void)
    {
        base::print();
    }
};


int main()
{
    derived a;
    a.print();

    return 0;
}
```

下面代码显示了不同调用虚函数时的实际执行情况：
```
#include <iostream>
using namespace std;

class A
{
public:
    virtual void print(void)
    {
        cout << "A: print" << endl;    
    }
};

class B: public A
{
public:
    virtual void print(void)
    {
        cout << "B: print" << endl;    
    }
};

class C: public A
{
public:
    virtual void print(void)
    {
        cout << "C: print" << endl;    
    }
};

void print(A a)
{
    a.print();    
}

int main()
{
    A a, *pa, *pb, *pc;
    B b;
    C c;

    pa = &a;
    pb = &b;
    pc = &c;

    a.print();
    b.print();
    c.print();

    pa->print();
    pb->print();
    pc->print();

    print(a);
    print(b);
    print(c);

    return 0;
}
```

执行结果为:

>A: print                                                                                                                                                                           
>B: print                                                                                                                                                                           
>C: print                                                                                                                                                                           
>A: print                                                                                                                                                                           
>B: print                                                                                                                                                                           
>C: print                                                                                                                                                                           
>A: print                                                                                                                                                                           
>A: print                                                                                                                                                                           
>A: print
