---
layout: post
title:  C++虚函数表原理相关代码
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
using namespace std;

class Test{
public:
    void t(void){};
};

class Base {
public:
    virtual void f() {cout<<"base::f"<<endl;}
    virtual void g() {cout<<"base::g"<<endl;}
    virtual void h() {cout<<"base::h"<<endl;}

    int n;
};

class Derive : public Base{
public:
    void g() {cout<<"derive::g"<<endl;}
};


int main () {
    cout<<"size of Base: "<<sizeof(Base)<<endl;
    cout << "size of Test: " << sizeof(Test)<<endl;//Test类中没有虚函数，因此没有虚函数表

    typedef void(*Func)(void);
    Base b;
    Base *d = new Derive();

    Func f=NULL, g=NULL, h=NULL;

    if (sizeof(void*) == sizeof(int))
    {
        int* pvptr = reinterpret_cast<int*>(d);
        int* vptr = reinterpret_cast<int*>(*pvptr);
        f = (Func)vptr[0];
        g = (Func)vptr[1];
        h = (Func)vptr[2];
    }
    else if (sizeof(void*) == sizeof(long))
    {
        long* pvptr = reinterpret_cast<long*>(d);
        long* vptr = reinterpret_cast<long*>(*pvptr);  
        f = (Func)vptr[0];
        g = (Func)vptr[1];
        h = (Func)vptr[2];
    }
    else if (sizeof(void*) == sizeof(long long))
    {
        long long* pvptr = reinterpret_cast<long long*>(d);
        long long* vptr = reinterpret_cast<long long*>(*pvptr);
        f = (Func)vptr[0];
        g = (Func)vptr[1];
        h = (Func)vptr[2];
    }

    f();
    g();
    h();    

    return 0;
}
```

 下面代码侧面说明，编译器在构造函数末尾处初始化vptr。所以如果在构造函数中调用virtual函数，动态绑定不会起作用，只会调用本类中的virtual函数：

```
#include <iostream>
using namespace std;

class base
{
public:
    base()
    {
        doSth();
    }
    virtual void doSth(void)
    {
        cout << "I am Base" << endl;
    }
};

class derived: public base
{
public:
    virtual void doSth(void)
    {
        cout << "I am Derived" << endl;
    }
};

int main()
{
    derived b;
    return 0;
}
```

 下面代码是我自己写的读取虚函数表示例：

```
#include <iostream>
using namespace std;

class Base{
public:
    void func0(void)
    {
        cout << "Base::func0" <<endl;
    }
    virtual void func1(void)
    {
        cout << "Base::func1" << endl;
    }
    virtual void func2(void)
    {
        cout << "Base::func2" << endl;
    }

private:
    int num1;
    int num2;
};

class Derived: public Base
{
public:
    void func0(void) //当Base中该函数声明为virtual，Derived中不写virtual也一样
    {
        cout << "Derived::func0" <<endl;
    }
    virtual void func1(void)
    {
        cout << "Derived::func1" << endl;
    }
    virtual void func3(void)
    {
        cout << "Derived::func3" << endl;
    }
};

typedef void (*Fun)(void);
template <class T>
inline Fun vTblItem(Base & obj, int i, T)
{
    return (Fun)(*((T)(*(T)(&obj))+i));
}
Fun getVTblItem(Base & obj, int i)
{
    void *p = NULL;
    if (sizeof(void*) == sizeof(int))
    {
        return vTblItem(obj, i, (int*)p);
    }
    else if (sizeof(void*) == sizeof(long))
    {
        return vTblItem(obj, i, (long*)p);
    }
    else if (sizeof(void*) == sizeof(long long))
    {
        return vTblItem(obj, i, (long long*)p);
    }
}

int main()
{
    Derived b;

    getVTblItem (b,0)();
    getVTblItem (b,1)();
    getVTblItem (b,2)();

    Base & a = b;
    a.func0();

    return 0;
}
```

屏幕输出：
>Derived::func1
>Base::func2
>Derived::func3
>Base::func0  

如果将
```
class Base{
public:
    void func0(void)
　　... ...
```

改为

```
class Base{
public:
    virtual void func0(void)
　　... ...
```

则func0()也加入到了虚函数表中，并且是表中第1项。屏幕输出为：
>Derived::func0

>Derived::func1

>Base::func2

>Derived::func0
