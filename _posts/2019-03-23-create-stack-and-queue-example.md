---
layout: post
title:  "创建stack和queue示例代码"
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
#include <stack>
#include <queue>#include <list>

using namespace std;


int main()
{
    stack<int, vector<int>> s;
    queue<int, list<int>> q; // the default container of queue is deque

    for (int i = 0; i < 10; i++)
    {
        s.push(i);
        q.push(i);
    }
    cout << "Stack:" << endl;
    while (!s.empty())
    {
        cout << s.top() << endl;
        s.pop();
    }

    cout << "Queue:" << endl;
    while (!q.empty())
    {
        cout << q.front() << endl;
        q.pop();
    }

    while (1);

    return 0;
}

```
