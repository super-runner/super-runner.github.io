---
layout: post
title:  vector中的erase方法与algorithm中的remove有什么区别
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




```
/*
    erase()删除指定元素，元素个数减1，即size--
    remove()同时删除所有指定值的元素，其它元素前移补位，但元素总个数不变(size不变)
*/

#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

template<typename T>
void print (vector<T> &x)
{
    typename vector<T>::iterator it; //此处声明typename是为区别模板特例化，否则会编译错误
    for (it=x.begin(); it!=x.end(); it++)
    {
        cout << *it << " ";
    }
    cout << endl;
}

int main()
{
    vector<int> v;
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
    v.push_back(4);
    v.push_back(4);
    v.push_back(5);
    v.push_back(7);
    v.push_back(6);

    v.erase(v.begin());
    print(v);

    vector<int>::iterator pos;
    pos = remove(v.begin(), v.end(), 4);
    print(v);


    return 0;
}
```
