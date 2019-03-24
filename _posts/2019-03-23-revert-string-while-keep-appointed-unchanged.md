---
layout: post
title:  "自写代码：反转字符串，但保持指定子串不变"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





empty() 堆栈为空则返回真

pop() 移除栈顶元素

push() 在栈顶增加元素

size() 返回栈中元素数目

top() 返回栈顶元素

```
#include <iostream>
#include <string.h>
using namespace std;
char * strRev (char * src, unsigned int count)
{
    if (src == NULL) return NULL;
    char * start = src;
    char * end = src + count -1;
    while (start < end)
    {
        *start ^= *end;
        *end ^= *start;
        *start ^= *end;
        ++start;
        --end;
    }
    return src;
}
char * revStrKeepSub (char * src, char * sub)
{
    if ((src==NULL) || (sub==NULL)) return NULL;
    unsigned int aLen = strlen(src);
    unsigned int bLen = strlen(sub);
    strRev(src, aLen);
    strRev(sub, bLen);
    strRev(strstr(src, sub), bLen);
    strRev(sub, bLen);    
    return src;
}
int main()
{
    char a[] = "abcdefg";
    char b[] = "cdef";
    cout << revStrKeepSub (a, b) << endl;
    return 0;
}
```
