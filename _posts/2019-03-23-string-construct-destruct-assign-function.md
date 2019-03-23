---
layout: post
title:  "C++ String类的构造、析构和赋值函数"
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

class String
{
public:
    String(const char * str=NULL)
    {
        if (str == NULL)
        {
            //这两行代码保证了：
            //任一String对象的m_string都不会是NULL，它指向heap上分配给它的空间
            m_string = new char[1];
            m_string = '\0';
        }
        else
        {
            m_string = new char[strlen(str)+1];
            strcpy (m_string, str);
        }
    }

    String(const String & s)
    {
        m_string = new char[strlen(s.m_string)+1];
        strcpy (m_string, s.m_string);
    }

    ~String()
    {
        if (m_string != NULL)
        {
            delete [] m_string;
            m_string = NULL;
        }

        cout << "destructor: m_string is " << ((m_string)?(m_string):("NULL")) << endl;
    }

    String & operator=(const String & s)
    {
        if (&s != this) //注意：判断是否同一对象，要对对象的地址比较
        {
            delete [] m_string;
            m_string = new char[strlen(s.m_string)+1];
            strcpy (m_string, s.m_string);
        }

        return *this;
    }

    char * str(void)
    {
        return m_string;
    }

private:
    char * m_string;
};

int main()
{
    String * a = new String("Becky");
    String * b = new String(NULL);
    String c;

    c= *b = *a;
    cout << "b = " << b->str() <<endl;
    cout << "c = " << c.str() << endl;

    delete a;
    delete b;

    return 0;

}
```
