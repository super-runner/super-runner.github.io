---
layout: post
title:  "C++ vector示例代码"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> array;
    array.push_back(1);
    array.push_back(2);
    array.push_back(3);
    array.push_back(2);
    array.push_back(4);
    array.push_back(2);

    cout << "Before erase all 2" << endl;
    for (vector<int>::iterator i = array.begin(); i != array.end(); i++)
    {
        cout << *i << endl;
    }

    vector<int>::iterator i = array.begin();
    while(i != array.end())
    {
        if (*i == 2)
        {
            i = array.erase(i);
        }
        else
        {
            ++i;
        }
    }

    cout << "After erase all 2" << endl;
    for (vector<int>::iterator i = array.begin(); i != array.end(); i++)
    {
        cout << *i << endl;
    }

    while (1);

    return 0;
}
```
