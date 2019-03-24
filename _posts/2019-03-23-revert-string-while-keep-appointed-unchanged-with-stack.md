---
layout: post
title:  "自写代码：反转字符串，但指定子串不反转 （使用了stack)"
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
#include <stack>
using namespace std;
char * revStrExcludeSub (char * src, char * sub)
{
    if ((src==NULL) || (sub==NULL)) return NULL;
    char * head=src, * tail=src, * pSub=sub;
    stack<char> st;
    while (*head != '\0')
    {
        while ((*head==*pSub) && (*pSub!='\0'))
        {
            ++head;
            ++pSub;                
        }
        // find one match with sub string
        if (*pSub == '\0')
        {
            char * p = head-1;
            while (p >= tail)
            {
                // cout << "1 push: " << *p <<endl;    
                st.push(*p--);
            }
            tail = head;
            pSub = sub;
        }
        else
        {
            st.push(*tail);
            // cout << "2 push: " << *tail <<endl;
            tail++;
            head = tail;
            pSub = sub;
        }
    }
    head = src;
    while (!st.empty())
    {
        *head++ = st.top();
        st.pop();
    }
    *head = '\0';
    return src;
}
int main()
{
    char a[] = "123434578345678";
    char b[] = "345";
    cout << "input:  " << a << " - sub string: " << b << endl;
    cout << "output: " << revStrExcludeSub (a, b) << endl;
    return 0;
}

```
