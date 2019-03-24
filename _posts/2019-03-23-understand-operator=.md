---
layout: post
title:  对 "operator="调用的理解
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}






A & operator= (A & b) {}
A obj1;
A obj2(1,2);

obj1 = obj2; //这条赋值语句可理解如下：
obj1.=(obj2) , 相当于obj1调用了它自己的=(A &b) 函数, 而=就是函数名。

因此obj1=obj2这句函数调用本身是有返回值的，类型是A &, 这就是为什么可以实现链式赋值: obj3 = obj1 = obj2;
