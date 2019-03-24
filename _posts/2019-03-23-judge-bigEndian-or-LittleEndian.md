---
layout: post
title:  "自写代码：判断CPU是Big Endian还是Little Endian"
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
bool checkLittleEnd (void)
{
    union test
    {
        int a;
        char b;
    } t;
    t.a = 1;
    return (t.b==1);
}
int main()
{
    if (checkLittleEnd())
        cout << "Little End!" << endl;
    else
        cout << "Big End!" << endl;
    return 0;
}

```
