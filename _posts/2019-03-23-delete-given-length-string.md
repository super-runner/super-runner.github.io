---
layout: post
title:  "自写代码：删除指定长度的字符"
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
// pos starting from 0
char * deleteChars1 (char * src, int pos, int len)
{
    if ((src==NULL) || (pos<0) || (len<=0)) return NULL;
    char * p = src;
    while (*p++ != '\0');
    p -= 2;
    if ((src+pos>p) || (src+pos+len-1>p)) return NULL;
    memcpy (src+pos, src+pos+len, (p+1)-(src+pos+len));
    *(p+1-len) = '\0';
    return src;
}
char * deleteChars2 (char * src, int pos, int len)
{
    if ((src==NULL) || (pos<0) || (len<=0)) return NULL;
    int srcLen = strlen(src);
    if ((pos>=srcLen) || (pos+len>srcLen)) return NULL;
    char * p = src+pos;
    while(*(p+len) != '\0')
    {
        *p = *(p+len);
        p++;
    }
    *p = '\0';
    return src;
}
int main()
{
    char a[] = "12345678";
    cout << deleteChars2(a, 3, 4) << endl;
    return 0;
}

```
