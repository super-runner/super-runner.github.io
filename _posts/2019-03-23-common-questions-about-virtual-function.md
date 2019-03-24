---
layout: post
title:  关于虚函数的一些常见问题
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





[引用此处](https://www.cnblogs.com/burellow/archive/2011/05/25/2056506.html)

1) 虚函数是动态绑定的，也就是说，使用虚函数的指针和引用能够正确找到实际类的对应函数，而不是执行定义类的函数。这是虚函数的基本功能，就不再解释了。

2) 构造函数不能是虚函数。而且，在构造函数中调用虚函数，实际执行的是父类的对应函数，因为自己还没有构造好, 多态是被disable的。

3) 析构函数可以是虚函数，而且，在一个复杂类结构中，这往往是必须的。

4) 将一个函数定义为纯虚函数，实际上是将这个类定义为抽象类，不能实例化对象。

5) 纯虚函数通常没有定义体，但也完全可以拥有。

6)  析构函数可以是纯虚的，但纯虚析构函数必须有定义体，因为析构函数的调用是在子类中隐含的。

7) 非纯的虚函数必须有定义体，不然是一个错误。

8) 派生类的override虚函数定义必须和父类完全一致。除了一个特例，如果父类中返回值是一个指针或引用，子类override时可以返回这个指针（或引用）的派生。例如，在上面的例子中，在Base中定义了 virtual Base* clone(); 在Derived中可以定义为 virtual Derived* clone()。可以看到，这种放松对于Clone模式是非常有用的。


```
#include <iostream>
#include <string.h>
using namespace std;

class Base
{
public:
virtual ~Base(){} // virtual, not pure
virtual void test() const = 0; // pure virtual
};

void Base::test() const // 纯虚函数也可以有定义体
{
cout << "Base::test()" << endl;
}

class Derived: public Base
{
public:
Derived(){}
virtual void test () const
{
cout << "Derived::test()" << endl;
}

virtual void foo(){}
};

int main()
{
Base *pb = new Derived();
pb->test();
pb->Base::test(); // 可强制指定调用Base类中的test()

return 0;
}
```
