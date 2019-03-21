---
layout: post
title:  "函数对象 function objects"
date: 2019-01-05 20:27 +0800
categories: c++
tags:  programming
author: Jason Chi
---
函数对象，就是一个重载了"()"运算符的类的对象，它可以像一个函数一样使用。

STL中提供了一元和二元函数的两种函数对象：(都是模板）
一元函数对象：
>negate, 相反数

>logical_not, 逻辑非(!)

二元函数对象:

- plus, 加法
minus, 减法
multiplies, 乘法
divides, 除法
modulus, 求余
equal_to, 等于(==)
not_equal_to, 不等于(!=)
greater, 大于(>)
greater_equal, 大于或等于(>=)
less, 小于(<)
less_equal, 小于或等于(<=)
logical_and, 逻辑与(&&)
logical_or, 逻辑或(||)
```

```
