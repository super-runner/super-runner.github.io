---
layout: post
title:  "C++的空类默认会产生哪些类成员函数"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
class EMPTY
{
public:
    EMPTY(); //默认构造函数
    EMPTY(const EMPTY &); //复制构造函数
    ~EMPTY();//默认析构函数
    EMPTY & operator=(const EMTPY &); //赋值运算符
    EMPTY * operator&();  //取址运算符
    const EMPTY * operator&() const; //取址运算符 const
};

```
