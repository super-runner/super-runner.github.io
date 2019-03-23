---
layout: post
title:  "自写代码：找出01字符串中0和1连续出现的最大次数"
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
void calculate (const char * src, int * max0, int * max1)
{
    if (src == NULL) return;
    int tmpCnt=1;
    *max0 = 1;
    *max1 = 1;
    while (*src++)
    {
        if (*src == *(src-1))
        {
            tmpCnt++;
            if (*src=='0' && *max0 < tmpCnt) *max0 = tmpCnt;
            if (*src=='1' && *max1 < tmpCnt) *max1 = tmpCnt;
        }
        else
        {
            tmpCnt = 1;    // pay attention : 1 not 0
        }
    }
}
int main()
{
    char a[] = "100100000001111000111111111000";
    int cnt0 = 0;
    int cnt1 = 0;
    calculate(a, &cnt0, &cnt1);
    cout << "0: " << cnt0 << " - 1: " << cnt1 << endl;
    return 0;
}

```
