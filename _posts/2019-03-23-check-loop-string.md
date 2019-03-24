---
layout: post
title:  "查看字符串是否是回文"
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
bool checkRevStr (const char * src)
{
    if (src == NULL) return false;
    const char * end = src + strlen(src) - 1;
    while (*src++ == *end--);
    return (src<end)?(false):(true);
}
int main()
{
    char a[] = "level";
    char b[] = "hanna";
    if (checkRevStr(b))
    {
        cout << "yes, it is!" << endl;
    }
    else
    {
        cout << "No, it is not!" << endl;
    }
    return 0;
}

```
