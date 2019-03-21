---
layout: post
title: "c++实现bind1st和bind2nd函数适配器"
categories: c++
tags:  programming
author: Jason Chi
---

* content
{:toc}




```
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

template<class Operation, class Para>
class my_binder1st
{
public:
    my_binder1st(const Operation op, const Para first): m_op(op), m_first(first)
    {
    }
    Para operator()(Para second)
    {
        return m_op(m_first, second);
    }

private:
    Operation m_op;
    Para m_first;
};

template<class Operation, class Para>
my_binder1st<Operation, Para> my_bind1st(Operation op, Para first)
{
    return my_binder1st<Operation, Para>(op, first);
}

template<class Operation, class Para>
class m_binder2nd
{
public:
    m_binder2nd(Operation op, Para second):m_op(op), m_second(second)
    {
    }
    Para operator()(Para first)
    {
        return m_op(first, m_second);
    }

private:
    Operation m_op;
    Para m_second;
};

template<class Operation, class Para>
m_binder2nd<Operation, Para> m_bind2nd (Operation op, Para second)
{
    return m_binder2nd<Operation, Para>(op, second);
}

int main()
{
    // bind 1st parameter
    my_binder1st<plus<int>, int> plusObj1(plus<int>(), 10);
    cout << plusObj1(20) << endl;

    my_binder1st<plus<int>, int> plusObj2 = my_bind1st(plus<int>(), 20);
    cout << plusObj2(15) << endl;

    // bind 2nd parameter
    m_binder2nd<minus<double>, double> minusObj1(minus<double>(), 5.2);
    cout << minusObj1(27.6) << endl;

    // classical usage: count_if
    vector<int> v;
    for (int i=1; i<10; i++)
    {
        v.push_back(i);
    }
    cout << count_if(v.begin(), v.end(), m_bind2nd(greater<int>(), 3)) << endl;
    return 0;
}
```
