---
layout: post
title:  "实现冒泡排序模板"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
using namespace std;

template<class T>
bool increSort(T &a, T &b)
{
    return (a > b);
}

template<class T>
bool decreSort(T &a, T &b)
{
    return (a < b);
}

template<class T>
void Sort(T* array, int len, bool(*compare)(T&, T&))
{
    T temp;
    for (int i = 0; i < len - 1; i++)
    {
        for (int j = len - 1; j > i; j--)
        {
            if (compare(array[j-1], array[j]))
            {
                temp = array[j];
                array[j] = array[j - 1];
                array[j - 1] = temp;
            }
        }
    }
}

template<class T>
class Rec
{
public:
    Rec(T a = 0, T b = 0) :length(a), width(b) {}
    T Area()
    {
        return length * width;
    }

private:
    T length;
    T width;
};

template<class T>
bool operator > (Rec<T> & a, Rec<T> &b)
{
    return (a.Area() > b.Area());
}

template<class T>
bool operator < (Rec<T> & a, Rec<T> &b)
{
    return (a.Area() < b.Area());
}

int main()
{
    Rec<double> a[] =
    {
        Rec<double>(1.1, 5.2),
        Rec<double>(3.3, 7.8),
        Rec<double>(4.4,2.8),
        Rec<double>(6.1,1.5)
    };
    int size = sizeof(a) / sizeof(Rec<double>);

    cout << "size is " << size << endl;

    Sort(a, size, increSort);

    for (int i = 0; i < size; i++)
    {
        cout << a[i].Area() << endl;
    }


    int i;
    cin >> i;

    return 0;
}

```
