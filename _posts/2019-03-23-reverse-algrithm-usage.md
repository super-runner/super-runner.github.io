---
layout: post
title:  reverse算法的应用
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




```
#include <iostream>
#include <string>
#include <algorithm>
#include <vector>
#include <list>
#include <deque>
using namespace std;

int main()
{
    // demonstrate reverse an int array
    int a[4] = {3,2,4,1};

    reverse<int [4]>(a,a+4);

    cout << "After reverse int a[4]:" << endl;
    for (int i=0; i<4; i++)
    {
        cout << a[i] << endl;
    }

    // demonstrate reverse a vector
    vector<string> v;
    v.push_back("Jason");
    v.push_back("Becky");
    v.push_back("Jessie");
    v.push_back("Alex");

    reverse<vector<string>::iterator>(v.begin(), v.end());

    cout << "After reverse vector v:" << endl;
    for (vector<string>::iterator i=v.begin(); i!=v.end(); i++)
    {
        cout << (*i) << endl;
    }

    // demonstrate reverse a list
    list<int> l;
    l.push_back(10);
    l.push_back(20);
    l.push_back(30);
    l.push_back(40);
    reverse<list<int>::iterator>(l.begin(), l.end());
    for (list<int>::iterator i=l.begin(); i!=l.end(); i++)
    {
        cout << (*i) << endl;
    }

    // demonstrate reverse a deque
    deque<int> d;
    d.push_back(100);
    d.push_back(200);
    d.push_back(300);
    d.push_back(400);
    reverse<deque<int>::iterator>(d.begin(), d.end());
    for (deque<int>::iterator i=d.begin(); i!=d.end(); i++)
    {
        cout << (*i) << endl;
    }

    return 0;
}
```
