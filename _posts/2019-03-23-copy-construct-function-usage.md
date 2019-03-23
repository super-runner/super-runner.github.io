---
layout: post
title:  "C++复制构造函数应用场景"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




复制构造函数应用场景：

1.一个对象以值传递的方式传入函数体

2.一个对象以值传递方式从函数返回

3.一个对象需要通过另外一个对象进行初始化

```
#include <iostream>
#include <string.h>
using namespace std;
class Test
{
public:
    int a;
    Test(int x):a(x){}
    Test(const Test &test)  // 复制构造函数
    {
        cout << "copy constructor" << endl;
        a = test.a;
    }
    ~Test()
    {
        cout << "destructor a = " << a << endl;
    }
};
void func1(Test t)           // 1.值传递传入函数体
{
    cout << "func1()..." << endl;
}
Test func2()                 // 2.值传递从函数体返回
{
    Test t(2);
    cout << "func2()..." << endl;
    return t;
}
int main()
{
    Test t1(1);
    Test t2 = t1;            // 3.用t1对t2做初始化
    cout << "before func1() ... " << endl;
    func1(t1);
    Test t3 = func2();
    cout << "after func2() ... " << endl;
    return 0;
}
```
