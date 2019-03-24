---
layout: post
title:  "自写代码：在字符串中插入连续字符的个数"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





例如字符串aabbbc，插入字符个数后变成aa2bbb3c1

```
#include <iostream>
#include <string.h>
using namespace std;
char * strInsertCount (const char * str)
{
    if (str==NULL) return NULL;
    char * retStr = new char[strlen(str)*2+1];
    char * p = retStr;
    int count = 1;
    while ((*p++=*str++) != '\0')
    {
        if (*str==*(str-1))
        {
            count++;
        }
        else
        {
            *p++ = count + '0';
            count = 1;
        }
    }
    return retStr;
}
int main()
{
    char a[] = "aadiiiisasbbbeewww";
    char * p = strInsertCount (a);
    cout << "Input:  " << a <<endl;
    cout << "Output: " << p << endl;
    delete [] p;
    return 0;
}
```
