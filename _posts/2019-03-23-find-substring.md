---
layout: post
title:  "自写代码：查找子字符串"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <assert.h>
#include <string.h>
using namespace std;
const char * strstr1 (const char * src, const char * des)
{
    assert ((src!=NULL)&&(des!=NULL));
    int srcLen = strlen(src);
    int desLen = strlen(des);
    if ((srcLen<desLen) || (srcLen==0) || (desLen==0))
    {
        return NULL;
    }    
    const char * p = NULL;
    const char * q = NULL;
    int count = srcLen - desLen + 1;
    while (*src != '\0')
    {
        p = src;
        q = des;
        while (*p++ == *q++)
        {
            if (*q == '\0') return src;  
        }
        if (--count == 0) return NULL;
        src++;
    }
    return NULL;
}
int main ()
{
    char a[] = "012345678905";
    char b[] = "7890";
    cout << strstr1(a, b) << endl;
    return 0;
}

```
