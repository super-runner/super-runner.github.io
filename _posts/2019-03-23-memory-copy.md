---
layout: post
title:  "自写代码：内存copy"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <assert.h>
using namespace std;
void * memcpy1 (void * desAddr, const void * srcAddr, unsigned int count)
{
    assert ((desAddr!=NULL) && (srcAddr!=NULL));
    char * from = NULL;
    char * to   = NULL;
    if ((desAddr==srcAddr) || (count==0))
    {
        return NULL;
    }
    if (desAddr < srcAddr)
    {
        from = (char*)srcAddr;
        to = (char*)desAddr;
        while (count-- > 0)
        *to++ = *from++;
    }
    else
    {
        from = (char*)(srcAddr + count-1);
        to = (char*)(desAddr + count-1);
        while (count-- > 0)
        *to-- = *from--;
    }
    return desAddr;
}
int main()
{
    int * to = (int*)malloc(sizeof(int) * 200);
    int * from = to + 50;
    for (int i=0; i<100; i++)
    {
        from[i] = i+1;
    }
    if (memcpy1 ((void*)to, (void*)from, sizeof(int)*100) == NULL)
    {
        cout << "something wrong!" << endl;
    }
    for (int i=0; i<100; i++)
    cout << to[i] << endl;
    free((from>to)?(to):(from));
    return 0;
}

```
