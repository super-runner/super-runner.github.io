---
layout: post
title:  C++静态成员变量的初始化
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
    static int i;                 // 这里不可以初始化非const static变量！
    static const int j = 1; // 1st place to initiate for const static variable
};
int demo::i = 0;       // only place to initiate for non-const static variable
//const int demo::j = 1; // 2nd place to initiate for const static variable
int main()
{
    return 0;
}
```
