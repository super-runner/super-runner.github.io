---
layout: post
title:  "自写代码：在字符串A中将子串B替换为子串C"
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
char * replace (char * in, char * s1, char * s2, char *out)
{
    if ((in==NULL)||(s1==NULL)||(s2==NULL)||(out==NULL)) return NULL;
    char *pOut = out;
    int lenS2 = strlen(s2);
    while (*in != '\0')
    {
        char * p = in;
        char * q = s1;
        while ((*p==*q) && (*q!='\0'))
        {
            p++;
            q++;
        }
        if (*q == '\0')
        {
            strcpy(pOut, s2);
            pOut += lenS2;
            in = p;
        }
        else
        {
            *pOut++ = *in++;
            p = in;
        }
        q = s1;
    }
    *pOut = '\0';
    return out;
}
int main()
{
    char a[] = "abcdefbcdkaldfbcdka";
    char b[] = "bcd";
    char c[] = "ooo";
    char out[50] = "";
    cout << replace (a, b, c, out) << endl;
    return 0;
}

```
