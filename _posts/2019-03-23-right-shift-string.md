---
layout: post
title:  "自写代码：实现字符串循环右移"
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
// not using string library
char * rightLoop1 (char * src, int n)
{
    if (src==NULL) return NULL;
    char * p = src;
    while (*p++);
    int len = p-1-src;
    n %= len;
    char * buff = new char [n];
    // move the overflowed into buffer
    for (int i=0; i<n; i++)
    {
        buff[i] = src[len-n+i];
    }
    // move the sring to the end
    for (int i=len-n-1; i>=0; i--)
    {
        src[i+n] = src[i];
    }
    // copy back from the buffer
    for (int i=0; i<n; i++)
    {
        src[i] = buff[i];
    }
    delete [] buff;
    return src;
}
// using strlen and memcpy
char * rightLoop2 (char * src, int n)
{
    if (src==NULL) return NULL;
    int len = strlen(src);
    n %= len;
    char * buff = new char [n];
    // move the overflowed into buffer
    memcpy (buff, src+len-n, n);
    // move the sring to the end
    for (int i=len-n-1; i>=0; i--)
    {
        src[i+n] = src[i];
    }
    // copy back from the buffer
    memcpy (src, buff, n);
    delete [] buff;
    return src;
}
int main()
{
    char a[] = "jason_becky";
    cout << rightLoop1 (a, 98) << endl;
    return 0;
}

```
