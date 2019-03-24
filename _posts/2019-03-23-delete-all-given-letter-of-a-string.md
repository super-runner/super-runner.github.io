---
layout: post
title:  "自写代码：删除字符串中所有指定的字符"
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
char * deleteChar (char * src, char c)
{
    if (src == NULL) return NULL;
    char * p = src;
    char * head = src;
    while (*p != '\0')
    {
        if (*p != c)
        {
            *head++ = *p;
        }
        p++;
    }
    *head = '\0';
    return src;
}
int main()
{
    char a[] = "132345336733333";
    cout << deleteChar(a, '3') << endl;
    return 0;
}

```
