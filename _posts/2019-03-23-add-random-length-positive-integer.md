---
layout: post
title:  "自写代码：实现任意长度的两个正整数相加"
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
int validateInt (const char * src)
{
    if (src == NULL) return 0;
    const char * p = src;
    while (*p>='0' && *p<='9') p++;
    return (*p=='\0')?(p-src):(0);
}
char * addBigInt (const char * x, const char * y)
{
    int xLen = validateInt(x);
    if (xLen == 0) return NULL;
    int yLen = validateInt(y);
    if (yLen == 0) return NULL;
    const char * a=NULL, *b=NULL;
    int aLen=0, bLen=0;
    // make sure a longth < b longth
    if (xLen < yLen)
    {
        a = x; b = y; aLen = xLen;bLen = yLen;
    }    
    else
    {
        a = y; b = x; aLen = yLen; bLen = xLen;
    }
    const char * pA = a, * pB = b;
    while (*pA++ != '\0');
    pA -= 2;  // pay attention: -2 since pA point to the next char of '\0'
    while (*pB++ != '\0');
    pB -= 2;
    int carry = 0, sum = 0;
    stack<char> st;
    while (pA >= a)
    {   
        sum = (*pA-'0'+*pB-'0'+carry)%10;
        carry = (*pA-'0'+*pB-'0'+carry)/10;
        st.push(sum+'0');
        pA--;
        pB--;
    }
    while (pB >= b)
    {
        sum = (*pB-'0'+carry)%10;
        carry = (*pB-'0'+carry)/10;
        st.push(sum+'0');        
        pB--;
    }
    if (carry) st.push('1');
    char * ret = new char [st.size()];
    char * p = ret;
    while (!st.empty())
    {
        *p++ = st.top();
        st.pop();
    }
    *p = '\0';
    return ret;
}
int main()
{
    char a[] = "6666";
    char b[] = "837653";
    cout << "a:  " << a << endl;
    cout << "b:" << b << endl;
    char * p = addBigInt (a, b);
    cout << "  " << p << endl;
    delete [] p;
    return 0;
}

```
