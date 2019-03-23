---
layout: post
title:  算法:find, for_each及reverse_iterator的应用
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}



```
#include <iostream>
#include <algorithm>
#include <deque>
using namespace std;

/*
    demonstrate algorithm: find, for_each
    demonstrate reverse_iterator
*/

void print (int x)
{
    cout << x << " ";
}

int main()
{
    deque<int> d;
    for (int i=1; i<10; i++)
    {
        d.push_back(i);
    }

    deque<int>::iterator pos1 = find(d.begin(), d.end(), 3);
    deque<int>::iterator pos2 = find(d.begin(), d.end(), 7);
    for_each(pos1, pos2, print);
    cout << endl;

    // when rpos1++, it moves backward instead of forward
    deque<int>::reverse_iterator rpos1(pos1);    
    deque<int>::reverse_iterator rpos2;
    rpos2 = (deque<int>::reverse_iterator)pos2;
    for_each(rpos2, rpos1, print); //左闭右开
    cout << endl;

    // 当为rpos1赋值为pos1时，rpos1实际指向了pos1的前一个元素
    cout << "pos1= " << *pos1 << ", rpos1: " << *rpos1 << endl;
    cout << "pos2= " << *pos2 << ", rpos2: " << *rpos2 << endl;

    return 0;
}
```
