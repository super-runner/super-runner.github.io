---
layout: post
title:  C++多重继承
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <complex>
using namespace std;

class Person
{
public:
    void eat(void)
    {
        cout << "Person->eat" << endl;
    }
};

class Author: virtual public Person
{
public:
    void write(void)
    {
        cout << "Auther->write" << endl;
    }
};

class Singer: virtualpublic Person
{
public:
    void sing(void)
    {
        cout << "Singer->sing" << endl;
    }
};

class Author_Singer: public Author, public Singer
{
public:
    void doSth(void)
    {
        cout << "AuthorSinger->doSth" << endl;
    }
};


int main()
{
    Author_Singer pp;
    pp.doSth();
    pp.write();
    pp.sing();
    pp.eat();
    //pp.Author::eat();

    return 0;
}
```

代码中Author和Singer以virtual方式继承Person，解决了Author_Singer实例拥有两份Person copy的问题，这是一种解决方法。

另一种方法是明确指定引用的子类：pp.Author::eat();

下面代码显示了，使用和不使用virtual来继承基类时，各个类的构造函数的执行情况
```
#include <iostream>

using namespace std;

class A
{
public:
    A()
    {
        cout << "A" << endl;
    }
};

class B: public A
{
public:
    B()
    {
        cout << "B" << endl;
    }
};

class C: public A
{
public:
    C()
    {
        cout << "C" << endl;
    }
};

class D: public C, public B
{
public:
    D()
    {
        cout << "D" << endl;
    }
};

int main()
{
    D d;

    return 0;
}
```

执行输出结果是：

>A

>C

>A

>B

>D

当类B, C采用virtual继承A后，代码如下
```
#include <iostream>
using namespace std;

class A
{
public:
    A()
    {
        cout << "A" << endl;
    }
};

class B: virtual public A
{
public:
    B()
    {
        cout << "B" << endl;
    }
};

class C: virtual public A
{
public:
    C()
    {
        cout << "C" << endl;
    }
};

class D: public C, public B
{
public:
    D()
    {
        cout << "D" << endl;
    }
};

int main()
{
    D d;

    return 0;
}
```

执行结果为：

>A

>C

>B

>D

** 注意：构造函数的执行顺序与继承列表中顺序一致 **
