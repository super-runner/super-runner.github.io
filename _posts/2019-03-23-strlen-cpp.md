---
layout: post
title:  实现strlen.cpp
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
unsigned int strlen1(const char * str)
{
    const char * p = str;
    while (*p++ != '\0');
    return (p-1-str);
}
int main()
{
    char a[] = "12345";
    cout << strlen1(a) << endl;
    return 0;
}

```
