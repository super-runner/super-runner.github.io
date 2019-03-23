---
layout: post
title:  "选用vector与 deque"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





Vector:

动态数组，内存中一整块连续区域。支持.reserve()和.capacity()。为提高效率，最好在添加元素之前用.reserve()分配好容量。插入删除操作越靠近数组首部效率越低。



deque(double ended queue):

动态数组，内存中多段连续区域拼凑。不支持reserve()和capacity()函数。首尾插入删除操作相对效率高。



总结：

vector和deque都是动态数组类型的，都可通过[]访问。一般选择vector, 但当需要从首尾两端进行插入或删除元素操作时，应该选择deque.
