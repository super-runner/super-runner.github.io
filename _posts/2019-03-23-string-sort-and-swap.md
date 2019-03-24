---
layout: post
title:  自写代码：字符串的排序及交换
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





将字符串分成两部分，前半部分按ASCII码升序排序，后半部分不变（如果字符串是奇数个，则中间的字符不变）再将前后两部分交换，最后将该字符串输出。

```
#include <iostream>
#include <string.h>
#include <list>
using namespace std;
// use list to sort
char * mySort1 (char * src)
{
    if (src == NULL) return NULL;
    char * p = src;
    int len = strlen(src);
    list<char> myList;
    for (int i=0; i<len/2; i++)
        myList.push_back(src[i]);
    myList.sort();
    if (len%2)
    {
        myList.push_front(src[len/2]);  
        for (int i=len-1; i>=len/2+1; i--)
            myList.push_front(src[i]);
    }
    else
    {
        for (int i=len-1; i>=len/2; i--)
            myList.push_front(src[i]);   
    }
    int i = 0;
    for (list<char>::iterator it=myList.begin(); it!=myList.end(); it++, i++)
    {
        src[i] = *it;
    }
    return src;
}
int main()
{
    char a[] = "428793615";
    cout << mySort1 (a);
    return 0;
}
```
