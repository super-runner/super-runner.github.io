---
layout: post
title:  自写MyString类：重载操作符 '+'
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

class MyString
{
public:
    MyString(const char * s=NULL)
    {
        if (s == NULL)
        {
            m_string = new char[1];
            m_string[0] = '\0';
        }
        else
        {
            m_string = new char[strlen(s)+1];
            strcpy(m_string, s);  
        }

    }

    ~MyString()
    {
        cout << "destructor: " << m_string << endl;
        delete [] m_string;
        m_string = NULL;
    }

    MyString & operator= (const MyString & s);
    MyString & operator+ (const MyString & s);
    friend MyString & operator+ (const MyString & left, const MyString & right);
    char * str(void);

//private:  // 为了支持上面的friend 操作符复用& ，需要将m_string开放的public，否则应为private。
    char * m_string;
};

MyString & MyString::operator= (const MyString & s)
{
    if (this == &s)
        return *this;

    if (m_string != NULL) // 这句其实可以不要，因为构造函数保证了任何对象的m_string都不会是NULL
        delete [] m_string;

    m_string = new char[strlen(s.m_string)+1];
    strcpy (m_string, s.m_string);
}


MyString & MyString::operator+ (const MyString & s)
{
    cout << "class: " << endl;
    if (strlen(s.m_string) == 0)
    {
        return *this;
    }

    MyString * p = new MyString;
    delete [] p->m_string;
    p->m_string = new char[strlen(m_string)+strlen(s.m_string)+1];

    strcpy (p->m_string, m_string);
    strcat (p->m_string, s.m_string);

    return *p;
}

// 强制重载操作符 &，用做加号
MyString & operator+ (const MyString & left, const MyString & right)
{
    cout << "friend: " <<endl;
    MyString * p = new MyString;
    delete [] p->m_string;
    p->m_string = new char[strlen(left.m_string)+strlen(right.m_string)+1];

    strcpy (p->m_string, left.m_string);
    strcat (p->m_string, right.m_string);

    return *p;
}

char * MyString::str(void)
{
    return m_string;
}

int main()
{

    MyString a("jason");
    MyString b("becky");

    MyString & c = a + b; // 当既定义了成员函数，也定义了普通函数（类外），类的成员函数会被执行。注释掉成员函数后，普通函数才会被调用。

    cout << "c:" << c.str() << endl;

    delete &c;


    return 0;
}
```
