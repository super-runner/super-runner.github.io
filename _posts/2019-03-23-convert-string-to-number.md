---
layout: post
title:  "自写代码：字符串转数字"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
include <iostream>
#include <string.h>
using namespace std;
bool strToInt (char * strIn, int * valueOut)
{
    if ((strIn==NULL) || (valueOut==NULL))
    {
        return false;
    }
    bool status = false;
    int  value = 0;
    int  i = (strIn[0]=='-')?(1):(0);
    while (strIn[i]>='0' && strIn[i]<='9')
    {
        value = value*10 + strIn[i] - '0';
        i++;
    }
    status = (i==strlen(strIn))?(true):(false);
    if (status)
    {
        *valueOut = (strIn[0]=='-')?(0-value):(value);
    }
    else
    {
        valueOut = NULL;
    }
    return status;
}
int main()
{
    char s[20] = "";
    int value;
    cin.getline (s, 20);    
    if (strToInt(s, &value))
    {
        cout << "input: " << value << endl;
    }
    else
    {
        cout << "Failure!\n" << endl;
    }
    return 0;
}
```
