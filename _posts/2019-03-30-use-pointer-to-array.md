---
layout: post
title:  指向数组的指针的使用
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}



指向数组的指针是一个指针，使用时相当于一个二维数组的名：
* 声明： `int (*p)[N]; // p指向长度为N的数组，它的步进或跨度是N`
* 使用： `p[0][2]`

```
#include <stdio.h>
#include <iostream>
using namespace std;

int main()
{
    int (*p)[5] = NULL;
    int array[20];

    for (int i=0; i<20; i++)
    {
        array[i] = i;
        cout << "array[" << i << "] = " << array[i] << endl;
    }

    p = reinterpret_cast<int (*)[5]>(array);
    //this also works: p = (int (*)[5])array;

    cout << "p[0][1] = " << p[0][1] << \
            ", p[1][2] = " << p[1][2] << \
            ", p[2][3] = " << p[2][3] << endl;

    return 0;
}

```
