---
layout: post
title:  "自写代码：设置或清除特定的位"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <stdio.h>
using namespace std;
#define BIT(n) (0x1<<n)
template <typename T>
T set_bit (T value, T bit_n)
{
    value |= BIT(bit_n);
    return value;
}
template <typename T>
T clear_bit (T value, T bit_n)
{
    value &= ~BIT(bit_n);
    return value;
}
int main()
{
    int a = 0;
    cout << "Please input a hex to set:" << endl;
    scanf("%x", &a);
    printf ("before: a = 0x%x\n", a);
    printf ("after : a = 0x%x\n", set_bit(a, 3));
    cout << "Please input a hex to clear:" << endl;
    scanf("%x", &a);
    printf ("before: a = 0x%x\n", a);
    printf ("after : a = 0x%x\n", clear_bit(a, 3));
    return 0;
}
}

```
