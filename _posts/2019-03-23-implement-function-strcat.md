---
layout: post
title:  "自写代码：实现库函数strcat()"
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
char * strcat1 (char * dest, char * src)
{
if ((dest==NULL) || (src==NULL)) return NULL;
char * start = dest;
while(*dest++ != '\0');
dest--;
while ((*dest++=*src++) != '\0');
return start;
}
int main()
{
char a[10] = "123";
char b[] = "456";
cout << strcat1(a, b) << endl;
}
```
