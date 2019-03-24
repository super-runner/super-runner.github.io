---
layout: post
title:  "自写代码：计算含有汉字的字符串长度"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




汉字作为一个字符处理，已知：汉字编码为双字节，其中首字节<0，尾字节在0~63以外（如果一个字节是 -128 ~ 127)

```
#include <iostream>
#include <string.h>
using namespace std;
int gbk_strlen(char * src)
{
    char * p = src;
    while (*p != '\0')
    {
        if ((*p<0) && (*(p+1)<0 || *(p+1)>63))
        {
            src++;
            p += 2;
        }
        else
        {
            p++;
        }
    }
    return (p - src);
}
int main()
{
    char * p = "abc我们de加油加油!";
    cout << gbk_strlen(p) << endl;
}
```
