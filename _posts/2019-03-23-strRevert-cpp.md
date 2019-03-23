---
layout: post
title:  实现strRevert.cpp
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
// rever letters of a string
char * strRevert1 (char * src)
{
    if (src == NULL) return NULL;
    int len = strlen(src);
    for (int i=0; i<len/2; i++)
    {
        src[i] = src[i] ^ src[len-1-i];
        src[len-1-i] = src[i] ^ src[len-1-i];
        src[i] = src[i] ^ src[len-1-i];
    }
    return src;
}
// revert words of a string - this solution is not good
char * strRevert2 (char * src)
{
    if (src == NULL) return NULL;
    int len = strlen(src);
    char * orgSrc = src;
    char des[len+1];
    des[len] = '\0';
    char * pDes = &des[len-1];
    char * pWord = orgSrc;
    int spaceCount = 0;
    if (*orgSrc == ' ') spaceCount = 1;
    while(1)
    {
        // end of a word or a ' ' set, copy the word or ' ' set into des[]
        if (((*orgSrc==' ')&&(!spaceCount)) || ((*orgSrc!=' ')&&(spaceCount)) || (*orgSrc=='\0'))
        {
            char * index = NULL;
            index = orgSrc - 1;
            do
            {
                *pDes = *index;
                pDes--;
                index--;
            }
            while (index >= pWord);
            if (*orgSrc=='\0') break;
            // prepare for a new word or new ' ' set
            pWord = orgSrc;
            spaceCount = (*orgSrc==' ')?(spaceCount+1):(0);
        }
        // continue with ' ' set
        else if ((*orgSrc==' ') && (spaceCount))
        {
            spaceCount++;
        }
        // continue with a word
        orgSrc++;
    }
    strcpy (src, des);
    return src;
}
// revert words of a string - this solution is better than strRevert2
char * strRevert3 (char * src)
{
    char * start=src, *end=src, *ptr=src;
    while (*ptr++ != '\0');
    end = ptr -2;
    while(start<end)
    {
        swap(*start++, *end--);
    }
    start = src;
    end = ptr -2;
    ptr = start;
    while (*ptr++ !='\0')
    {
        if (*ptr==' ' || *ptr=='\0')
        {
            end = ptr-1;
            while (start<end)
            swap(*start++, *end--);
            start = end = ptr+1;
        }
    }
return src;
}
int main()
{
    char a[19] = "who are you ?";
cout << "revert: " << strRevert3(a) << endl;
    cout << "revert: " << strRevert2(a) << endl;
    cout << "revert: " << strRevert1(a) << endl;
    return 0;
}

```
