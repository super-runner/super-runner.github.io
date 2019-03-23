---
layout: post
title:  C++动态绑定与静态绑定
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





只有虚函数才使用动态绑定，其它全部是静态绑定
编译阶段决定：non-virtual 函数->静态绑定->绑定静态类型
运行时决定：    virtual 函数->动态绑定->绑定动态类型


虚函数是动态绑定的，但是为了执行效率，缺省参数是静态绑定的。永远记住：
> 绝不重新定义继承而来的缺省参数（Never redefine function’s inherited default parameters value.
