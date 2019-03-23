---
layout: post
title:  函数对象 function objects
date: 2019-01-05 20:27 +0800
categories: c++
tags:  programming
author: Jason Chi
---

* content
{:toc}





函数对象，就是一个重载了"()"运算符的类的对象，它可以像一个函数一样使用。

STL中提供了一元和二元函数的两种函数对象：(都是模板）
一元函数对象：
* negate, 相反数
* logical_not, 逻辑非(!)

二元函数对象:

- plus, 加法
- minus, 减法
- multiplies, 乘法
- divides, 除法
- modulus, 求余
- equal_to, 等于(==)
- not_equal_to, 不等于(!=)
- greater, 大于(>)
- greater_equal, 大于或等于(>=)
- less, 小于(<)
- less_equal, 小于或等于(<=)
- logical_and, 逻辑与(&&)
- logical_or, 逻辑或(||)

下面是一段示例代码：

```
#include <iostream>
using namespace std;

template<class T>
class MyAdd
{
public:
    T operator()(T a, T b)
    {
        return a+b;
    }

};

template<class T>
class MyMin
{
public:
    T operator()(T a, T b)
    {
        return a-b;
    }
};

int main()
{
    MyAdd<double> obj1;
    cout << obj1(3.4,5.2) << endl;
    MyMin<int> obj2;
    cout << obj2(3, 7) << endl;

　　 //相反数
    negate<int> int_negate;
    cout << int_negate(5) << endl;
    //加法
    plus<int> int_plus;
    cout << int_plus(6, 7) << endl;
    //减法
    minus<int> int_minus;
    cout << int_minus(23, 8) << endl;
    //乘法
    multiplies<int> int_mult;
    cout << int_mult(3,7) << endl;
    //除法
    divides<int> int_divides;
    cout << int_divides(24, 4) << endl;
    //求余
    modulus<int> int_modulus;
    cout << int_modulus(23, 5) << endl;
    // ==
    equal_to<int> int_equal_to;
    cout << int_equal_to(3, 4) << endl;
    // !=
    not_equal_to<int> int_not_equal_to;
    cout << int_not_equal_to(3, 4) << endl;
    // >
    greater<int> int_greater;
    cout << int_greater(3, 4) << endl;
    // >=
    greater_equal<int> int_greater_equal;
    cout << int_greater_equal(3, 4) << endl;
    // <
    less<int> int_less;
    cout << int_less(3, 4) << endl;
　　 // <=
    less_equal<int> int_less_equal;
    cout << int_less_equal(3, 4) << endl;
    // &&
    logical_and<int> int_logical_and;
    cout << int_logical_and(0, 1) << endl;
    // ||
    logical_or<int> int_logical_or;
    cout << int_logical_or(0, 1) << endl;
    // !
    logical_not<int> int_logical_not;
    cout << int_logical_not(1) << endl;

    return 0;
}
```

 另一段示例代码：
 ```
 /*
 bind1st & bind2nd这两个函数模板都返回一元函数对象。bind1st绑定第1个参数，bind2nd绑定第2个参数。
 */

 #include <iostream>
 #include <algorithm>
 #include <vector>
 //#include <functional> //可以不引用
 using namespace std;

 template<class T>
 struct MyCompare: binary_function <T,T,bool>
 {
 public:
     bool operator()(const T &a, const T& b) const
     {
         return a<b;
     }
 };

 int main()
 {
     // demo bind1st
     binder1st<plus<int>> plusOjb = bind1st(plus<int>(), 10);
     cout << plusOjb(5) << endl;
     // demo bind2nd
     binder2nd<minus<double>> minusObj = bind2nd(minus<double>(), 10);
     cout << minusObj(30) << endl;

     // demo count_if with STL function object
     vector<int> v;
     for (int i=1; i<10; i++)
     {
         v.push_back(i);
     }
     cout << count_if(v.begin(), v.end(), bind2nd(greater<int>(), 3)) << endl;

     // demo count_if with self-written function object
     cout << count_if(v.begin(), v.end(), bind1st(MyCompare<int>(), 4)) << endl;

     return 0;
 }
 ```
