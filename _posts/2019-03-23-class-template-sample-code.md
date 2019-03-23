---
layout: post
title:  类模板sample code
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

template <class T1, class T2>
class Point
{
public:
   Point(T1 x=0, T2 y=0):a(x), b(y){}
   Point(const Point<T1,T2> & x)
   {
       a = x.a;
       b = x.b;
   }
   ~Point(){}

   Point<T1,T2> & operator= (const Point<T1,T2> & x)
   {
       a = x.a;
       b = x.b;
       return *this;
   }

   Point<T1,T2> operator+ (const Point<T1,T2> & x) //注意返回值不是引用而是值传递
   {
       Point<T1,T2> temp;
       temp.a = a + x.a;
       temp.b = b + x.b;
       return temp;
   }

   T1 a;
   T2 b;
};

template <class T1, class T2>
ostream & operator<< (ostream & out, Point<T1,T2> &x)
{
   out << "(" <<x.a << ", " << x.b << ") " << endl;
   return out;
}

int main()
{
   Point<int, double> a1(3, 2.4);
   Point<int,double> b1(2, 1.1);
   Point<int, double> c1;

   c1 = a1 + b1;
   cout << a1 << b1 << c1 << endl;

   Point<double, float> a2(1.11, 5.43);
   Point<double, float> b2(11.7,9.2);
   Point<double, float> c2 = a2 + b2;

   cout << a2 << b2 << c2 << endl;

   return 0;
}
```

下面代码展示通过继承方式把模板中与参数无关的代码分离出来
```
#include <iostream>
using namespace std;

template<class T>
class Base
{
public:
   Base(T x):value(x){}
   virtual void print(void)
   {
       cout << "Base::value = " << value << endl;
   }

   T value;
   int k=0;
};

template<class T, T num>
class Derived: public Base<T>
{
public:
   Derived():Base<T>(num){}
   virtual void print(void)
   {
       cout << "Derived::k = " << this->k << endl;
   }
};

int main()
{
   Derived<int, 1> a;
   Derived<int, 2> b;
   Derived<int, 3> c;
   a.print();
   b.print();
   c.print();

   return 0;
}
```
