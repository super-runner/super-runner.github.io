---
layout: post
title:  "没想通的关于复制构造函数C++代码: A obj=func()"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
class A
{
public:
    A(int x=0):i(x)
    {
        cout << "Normal Contructor: " << i << endl;
    }

    ~A()
    {
        cout << "Destructor: " << i << endl;
    }

    A(const A & x)
    {
        cout << "Copy Constructor: " << i << endl;
    }

    A & operator=(const A & x)
    {
        cout << "Operator = " << endl;
        return *this;
    }


    int i;

};

A fun ()
{
    A t(2);
    return t;
}
int main()
{
    A a(1);

    A b = fun();
    //fun();

    cout << "end of program" << endl;

    return 0;
}
  上面main()函数执行后的输出为：

  Normal Contructor: 1
  Normal Contructor: 2
  end of program
  Destructor: 2
  Destructor: 1

int main()
{
    A a(1);

    //A b = fun();
    fun();

    cout << "end of program" << endl;

    return 0;
}
```

输出为：
>Normal Contructor: 1
>Normal Contructor: 2
>Destructor: 2
>end of program
>Destructor: 1
