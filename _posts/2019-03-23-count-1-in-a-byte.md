---
layout: post
title:  "自写代码：计算一个字节中有多少位是1"
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
int calculateBits (unsigned char c)
{
    int count = 0;
    int bits = sizeof(c)*8;
    for (int i=0; i<bits; i++)
    {
        if (c & BIT(i)) count++;
    }
    return count;
}
void showBits (unsigned char c)
{
    printf ("Input:0x%x  ", c);
    int bits = sizeof(c)*8;
    for (int i=bits-1; i>=0; i--)
    {
        if (c & BIT(i))
            cout << "1";
        else
            cout << "0";
    }
    cout <<"b" << endl;
}
int main(int argc, char * agrv[])
{
    unsigned char c = 0;
    cout << "Please input a unsigned char in hex: " << endl;
    scanf ("%x", &c);
    showBits(c);
    cout << "bits of 1: " << calculateBits(c) << endl;
    return 0;
}

```
