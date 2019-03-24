---
layout: post
title:  "自写代码：demo一个类的创建"
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
class demo
{
public:
    static void showObjCount(void);
private:
    static int count;
    static const int j = 1;
public:
    void funcObj(void);
    demo(int x, int y);
    ~demo();
private:
    int k;
    const int m;
};
int demo::count = 0;
demo::demo(int x, int y):k(x), m(y)
{
    count++;
    cout << "Constructed: " << count << endl;
}
demo::~demo()
{
    count--;
    cout << "destructed: " << count << endl;
}
void demo::showObjCount(void)
{
    cout << "showObjCount: " << count << endl;
}
void demo::funcObj(void)
{
    cout << "k=" << k << " m=" << m << " count=" <<  count << " j=" << j <<endl;
}
int main()
{
    demo::showObjCount(); // 在不存在任何对象时即可直接调用类的方法
    demo * pObj = new demo(3, 4); // 在堆上创建对象的方式
    pObj->funcObj();      // 对象指针调用对象方法
    pObj->showObjCount(); // note:对象指针也可以调用类的方法
    demo obj(5,6);        // 在栈上创建对象的方式，作用域结束时自动执行析构函数
    obj.showObjCount();   //对象实例调用类方法
    delete pObj;
    return 0;
}

```
