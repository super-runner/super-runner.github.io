---
layout: post
title:  自己代码：实现字符串比较
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
int strcmp1 (const char * a, const char * b)
{
    int ret = 0;
    while (!(ret=*a-*b) && *b)
    {
        ++a;
        ++b;
    }
    return (ret>0)?(1):((ret<0)?(-1):(0));
}
int main ()
{
    const char a[] = "acde";
    const char b[] = "abcde";
    cout << "a - b = " << strcmp1(a,b) << endl;
    return 0;
}

```
