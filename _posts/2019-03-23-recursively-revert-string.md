---
layout: post
title:  "自写代码：递归实现字符串反转"
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
char * revStr (char * src, int len)
{
    if (len <= 1) return src;
    *src ^= *(src+len-1);
    *(src+len-1) ^= *src;
    *src ^= *(src+len-1);
    return (revStr(src+1,len-2)-1);
}
int main()
{
    char a[] = "abced";
    cout << revStr(a, strlen(a)) << endl;
    return 0;
}

```
