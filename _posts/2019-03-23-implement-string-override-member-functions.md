---
layout: post
title:  C++实现string各类运算符重载成员函数
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
    String(const char * x=NULL);
    String(const String & x);
    ~String();

    String & operator= (const String & x);
    String & operator= (const char * x);
    bool operator== (const String & x);
    bool operator!= (const String & x);
    bool operator< (const String & x);
    bool operator> (const String & x);
    String & operator+= (const String & x);
    String & operator+= (const char *x);
    char & operator[] (int i);
    char & operator() (const char * x);
    const char * str(void);
    unsigned int length(void);

    //friend函数可以任意定义到public, protect, or private
    //friend函数可以任意访问public, protected or private的数据
    friend ostream & operator<<(ostream & out, String & x);
    friend void test (String & x);
    friend void test (void);

private:
    char * m_string;
    unsigned int m_size;
    static int t; //test()测试用
};

int String::t = 0;

String::String(const char * x)
{
    if (x == NULL)
    {
        m_string = new char[1];
        m_string[0] = '\0';
        m_size = 0;
    }
    else
    {
        m_size = strlen(x);
        m_string = new char[m_size+1];
        strcpy (m_string, x);
    }
}
String::String(const String & x)
{
    m_size = x.m_size;
    m_string = new char[m_size+1];
    strcpy (m_string, x.m_string);
}
String::~String()
{
    cout << "destructor: " << m_string << endl;
    delete [] m_string;
    m_string = NULL;
    m_size = 0;

}
String & String::operator= (const String & x)
{
    if (this == &x)
        return *this;

    // 不需要判断m_string是否为NULL是因为它永远不会为NULL
    delete [] m_string;
    m_size = strlen(x.m_string);
    m_string = new char[m_size+1];
    strcpy (m_string, x.m_string);
    return *this;
}
String & String::operator= (const char * x)
{
    // 不需要判断m_string是否为NULL是因为它永远不会为NULL
    delete [] m_string;

    if (x == NULL)
    {
        m_string = new char[1];
        m_string[0] = '\0';
        m_size = 0;
    }
    else
    {
        m_size = strlen(x);
        m_string = new char[m_size+1];
        strcpy (m_string, x);
    }

    return *this;
}
bool String::operator== (const String & x)
{
    if (x.m_size != m_size)
        return false;

    for (int i=0; i<x.m_size; i++)
    {
        if (m_string[i] != x.m_string[i])
            return false;
    }

    return true;
}
bool String::operator!= (const String & x)
{
    return (*this==x)?(false):(true);
}
bool String::operator< (const String & x)
{
    int i = 0;
    while (m_string[i]==x.m_string[i] && m_string[i]!='\0')
    {
        i++;
    }
    return (m_string[i]-x.m_string[i]<0)?(true):(false);
}
bool String::operator> (const String & x)
{
    int i = 0;
    while (m_string[i]==x.m_string[i] && m_string[i]!='\0')
    {
        i++;
    }

    return (m_string[i]-x.m_string[i]>0)?(true):(false);
}
String & String::operator+= (const String & x)
{
    if (x.m_string == NULL)
        return *this;

    char * pOld = m_string;
    m_string = new char[m_size+x.m_size+1];
    strcpy (m_string, pOld);
    strcat (m_string, x.m_string);
    delete [] pOld;

    return *this;
}
String & String::operator+= (const char *x)
{
    if (x == NULL)
        return *this;

    char * pOld = m_string;
    m_string = new char[m_size+strlen(x)+1];
    strcpy (m_string, pOld);
    strcat (m_string, x);
    delete [] pOld;

    return *this;
}
char & String::operator[] (int i)
{
    return m_string[i];
}
char & String::operator() (const char * x)
{
    static char c = '?';
    if (!strcmp(x, "1st"))
        return m_string[0];

    if (!strcmp(x, "2nd"))
        return m_string[1];

    if (!strcmp(x, "3rd"))
        return m_string[2];
    else
        return c;
}
unsigned int String::length(void)
{
    return m_size;
}
const char * String::str(void)
{
    return m_string;
}
istream & operator>>(istream & in, String & x)
{
    char buf[50];
    in.getline(buf, 50);
    x = buf;
    return in;
}

ostream & operator<<(ostream & out, String & x)
{
    for (int i=0; i<x.length(); i++)
    {
        //out << x[i];
        out << x.m_string[i];
        /*
            如果在String类中声明本函数的friend,那么可以用out << x.m_string[i];
            代替 out << x[i];
        */
    }
    return out;
}

// used to test friend function acess static class member vairable
void test (void)
{
    cout << " String::t = " << String::t << endl;
    String::t++;
}
void test (String & x)
{
    cout << " x.t = " << x.t << endl;
    cout << "x.m_size = " << x.m_size << endl;    
    x.t++;
}

int main()
{
    String a = "jason";
    String b = a;
    a = "becky";
    a += b;
    b += "_added";

#if (0)
    cout << a("1st") <<a("3rd")<<a("2nd")<<a("kasf") << endl;
#endif

#if (0)
    cout << (a > b) << endl;
    cout << (a < b) << endl;
    cout << (a == b) << endl;
#endif

#if (1)
    test(a);
    test(a);
    test(b);
    test();
    test();
#endif

    return 0;
}
```
