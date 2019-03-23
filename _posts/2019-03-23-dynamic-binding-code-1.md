---
layout: post
title:  C++动态绑定(多态)example code 1
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <string.h>
using namespace std;

class Person
{
public:
    virtual void print(void)
    {
        cout << "I am a Person!" << endl;
    }
};

class Chinese: public Person
{
public:
    virtual void print(void)
    {
        cout << "I am a Chinese!" << endl;
    }
};

class American: public Person
{
public:
    virtual void print(void)
    {
        cout << "I am a American!" << endl;
    }
};

void printPerson (Person & person)
{
    person.print();
}

int main()
{
    Person p;
    Chinese c;
    American a;
    printPerson(p);
    printPerson(c);
    printPerson(a);

    return 0;
}
```
