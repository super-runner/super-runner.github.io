---
layout: post
title:  "自写代码：查找两个字符串的最大公共子串"
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
const char * findFirstLongestStr (const char * src, const char * des, unsigned int & count)
{
    if (src==NULL || des==NULL) return NULL;
    int desLen = strlen(des);
    char temp[desLen+1] = {'\0'}; // pay attention: +1 for '\0'
    count = 0;
    for (int len=desLen; len>0; len--)
    {
        for (int start=0; start<desLen-len+1; start++)
        {
            memcpy(temp, des+start, len);
            temp[len] = '\0';
            const char * where = strstr(src, temp);
            if (where)
            {
                count = len;
                return where;
            }
        }
    }
    return NULL;
}
int main()
{
    const char a[] = "kgalf3ehjnascdefgeodieha9duieadh";
    const char b[] = "abcdefg9023u1hdllis";
    unsigned int cn = 0;
    cout << cn << " : " << findFirstLongestStr(a, b, cn) << endl;
    return 0;
}

```
