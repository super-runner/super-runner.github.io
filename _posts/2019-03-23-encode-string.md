---
layout: post
title:  自写代码：字符串加密
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





1. 把字母替换成它后面的第4个字母。如：a->e, A->E, X->b, y->c
2. 翻转整个字符串

```
#include <iostream>
#include  <string.h>
using namespace std;
void strEncode(char * src)
{
    if (src == NULL) return;
    char * p = src;
    while (*p != '\0')
    {
        if (*p>='a' && *p<='z')
        {
            *p = (*p<'w')?(*p+4):('a'+*p+3-'z');
        }
        else if (*p>='A' && *p<='Z')
        {
            *p = (*p<'W')?(*p+4):('A'+*p+3-'Z');
        }
        p++;
    }
    --p;
    while (src < p)
    {
        *src ^= *p;
        *p ^= *src;
        *src ^= *p;
        ++src;
        --p;
    }
}
int main()
{
    char a[] = "a2Bc87DEf";
    cout << a << endl;
    strEncode(a);
    cout << a << endl;
    return 0;
}
```
