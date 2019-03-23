---
layout: post
title:  实现stack模板
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <list>

using namespace std;

template<class T>
class Stack
{
public:
    Stack()
    {
    }
    ~Stack()
    {
    }
    void push(T &e)
    {
        m_list.push_front(e);
    }
    void pop()
    {
        if (m_list.size() > 0)
        {
            m_list.pop_front();
        }
    }
    T top() const
    {
        return m_list.front();
    }
    int size()
    {
        return m_list.size();
    }
    bool empty()
    {
        return m_list.empty();
    }
    void clear()
    {
        m_list.clear();
    }

private:
    list<T> m_list;
};


int main()
{
    Stack<int> s;
    cout << s.empty() << endl;

    for (int i=0; i<10; i++)
        s.push(i);

    cout << s.empty() << endl;

    for (int i=0; i<10; i++)
    {
        cout << "top: " << s.top();
        s.pop();
        cout << " size:" << s.size() << endl;
    }

    cout << s.empty() << endl;

    return 0;
}

```
