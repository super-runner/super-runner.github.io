---
layout: post
title:  "C++复制构造函数的浅复制与深复制（避免浅复制引起的错误）"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




如果复制的对象中引用了某个外部的内容（例如分配在堆上的数据），那么在复制这个对象的时候，让新旧两个对象指向同一个外部的内容，就是浅复制；如果在复制这个对象的时候为新对象制作了外部对象的独立的COPY，就是深复制。

深拷贝和浅拷贝主要是针对类中的指针和动态分配的空间来说的，因为对于指针只是简单的值复制并不能分割开两个对象的关联，任何一个对象对该指针的操作都会影响到另一个对象。这时候就需要提供自定义的深拷贝的拷贝构造函数，消除这种影响。通常的原则是：

含有指针类型的成员或者有动态分配内存的成员都应该提供自定义的拷贝构造函数。在提供拷贝构造函数的同时，还应该考虑实现自定义的赋值运算符



下面代码中如果不自己定义复制构造函数：

编译器会自动生成默认的按位复制的复制构造函数。当类中定义了指针，并且在析构函数中释放指针所指向空间时，由于默认复制构造函数只是简单复制了指针的值，导致两个对象在调用析构函数时分别释放同一空间，产生多次释放的错误。

```
#include <iostream>
#include <string.h>
using namespace std;

class Test
{
public:
    Test(){buff = NULL;}
    Test(const char * str)
    {
        buff = new char[strlen(str)+1];
        strcpy(buff, str);
    }

    Test(const Test & v)
    {
        if (v.buff)
        {
            buff = new char[strlen(v.buff)+1];
            strcpy (buff, v.buff);
        }
        else
        {
            buff = NULL;
        }
    }

    ~Test()
    {
        if (buff)
        {
            delete buff;
        }
    }

    char * buff;
};

int main()
{
    Test t1("Hello world");
    Test t2 = t1;
    const char * p = (t1.buff==t2.buff)? "yes":"no";
    cout << p << endl;
    return 0;
}
```
