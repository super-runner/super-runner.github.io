---
layout: post
title:  "自写代码：复制字符串"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
using namespace std;
char * strcpy1 (char * strDest, char * strSrc)
{
    if ((strDest==NULL) || (strSrc==NULL))
    {
        return NULL;
    }
    char * strDeskCpy = strDest;
    while ((*strDest++ = *strSrc++) != '\0');
    return strDeskCpy;
}
int main()
{
    char a[] = "abcde";
    char b[20] = "";
    strcpy1(b, a);
    cout << "b: " << b << endl;
    return 0;
}

```
