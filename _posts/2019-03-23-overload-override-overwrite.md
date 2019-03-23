---
layout: post
title:  "[转] C++基础篇--overload重载&override覆盖&overwrite隐藏"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




[转自] (https://blog.csdn.net/ipmux/article/details/45038869)

Overload、Override和Overwrite英文接近，比较容易混淆，再加上翻译五花八门，使用时张冠李戴，往往是今天清楚明天糊涂。这三个概念在前面章节已分别讨论，这里再集中比较，以作备忘：
Overload （重载）

    前面分析过C++函数重载是借助C++的name mangling机制，允许在同一作用域中出现多个同名不同参的函数，如：

class Base{

  int output(int a) {……};

  int output(float b, float c){……};

}

  这是重载，特点：

  1）同名不同参：两个同名同参的函数绝不是重载，注意参数有无const也视为不同；

  2）仅仅返回值不同，不构成重载；

  3）相同作用域：同一文件域/全局域/namespace域内的同名不同参函数可以构成重载；而对于类成员函数，重载必须发生在同一类域中，分属于基类与派生类的函数不能直接构成重载（派生类函数重载基类函数的说法错误）。

Override （覆盖）

    在派生类中重新定义实现基类中的虚函数，这个过程就是override（译为覆盖）,如：

class Base{

  virtual void output() { cout << "Base::output()"<< endl;}

};

class Drv0 : public Base{

  void output () { cout << "Drv0:: output()" << endl; }

};

  这里Drv0::output就是覆盖（override）了Base::output（有人会错用重载overload），覆盖的特征：

    1）不同作用域（分别位于派生类与基类）；

    2）基类函数须为virtual函数，而派生类重新实现的函数需与基类虚函数同名同参。

Overwrite：隐藏（翻译最乱，也有称重定义、重写或屏蔽）

  指派生类的函数屏蔽与其同名的基类函数，发生在不同类域（派生类与基类之间），具体情形包括：

    1）派生类与基类函数同名但不同参，此时不论基类函数是否virtual，其都将被派生类函数隐藏。（不要与重载混淆，重载发生在同一类域）

    2）派生类函数与基类非virtual函数同名同参，此时基类函数被隐藏（不要与覆盖混淆，覆盖特指基类函数为virtual的情况)

    梳理后的逻辑：类public派生时，基类成员全部继承到派生类，除非派生类有同名；当派生类与基类成员有同名，除了基类成员为virtual函数时属于覆盖(override)，其他情况都是隐藏。

区分

    1) overload原理上与override/overwrite没有实质的关联和相似，overload是同一个类中的成员函数间的水平关系；而override/overwrite都是派生类和基类间的垂直关系，只要涉及派生类/基类函数间的关系就不可能是overload。只是overload译成重载，其字面含义与实际功能并不能形象匹配，反而暗示一种上下层间的垂直关系，导致派生类覆盖override或隐藏overwrite基类函数时常被误称为重载，因此只要留心，很容易把overload(重载)区别出来。

    2) override覆盖的前提或特点是基类函数必须为virtual，派生类中同名同参重新实现基类的virtual函数就是override，覆盖后可通过基类指针指向的具体对象类型确定调用哪个方法，从而实现函数多态；而overwrite(隐藏)是指派生类的函数屏蔽与其同名的基类非virtual函数，使其不能被派生类对象直接引用。

    下例说明覆盖与隐藏的区别：

class Base

{

  virtual void a(int x){  cout << "Base::a(int) "<< x << endl;  }

  void b(int x){  cout << "Base::b(int) "<< x << endl;  }

};

class Drv0 : public Base

{

  virtual void a(int x){  cout << "Drv0::a(int) "<< x << endl;  }

  void b(int x){  cout << "Drv0::b(int) "<< x << endl;  }  

};

void main()

{

  Drv0 d;

  Base *pbase = &d;

  Drv0 *pdrv = &d;



  pbase->a(3);  //输出Drv0::a(int) 3

  pdrv->a(3);  // 输出Drv0::a(int) 3



  pdrv->b(3); //  输出Drv0::b(int) 3

  pbase->b(3); // 输出Base::b(int) 3

}

    例中Drv0::a(int)覆盖override了Base::a(int)，因此不管是通过基类指针调用pbase->a(3)，还是通过派生类指针调用pdrv->a(3)，最终都是根据对象类型（Drv0 d;）选择调用函数，结果都为Drv0::a(int) 3

    而Drv0::b(int)则隐藏overwrite了Base::b(int)，外在表现就是根据指针类型（Base* pbase和Drv0* pdrv）决定调用函数，即pdrv->b(3)输出Drv0::b(int) 3，而pbase->b(3)输出Base::b(int) 3。因此隐藏相当于对于某一功能派生类另起炉灶，基类和派生类各干各的。

    所以覆盖的结果是根据对象类型决定调用哪个函数（多态）；而隐藏则表现为根据指针类型决定调用哪个函数，这是覆盖和隐藏最直观的区别。

    隐藏overwrite是对函数继承的破坏，多数情况没有必要，因为基类中非virtual型成员函数本就代表类的功能中不变的部分，派生类 应该直接继承使用而不需修改，发生overwrite多数说明设计不完善。

    实际上很多时候是不小心把override实现成了overwrite，即本该把基类函数设计为virtual以实现可变功能，却忘了写virtual，结果当在派生类中重写同名函数时就成了overwrite隐藏。
---------------------
作者：ipmux
来源：CSDN
原文：https://blog.csdn.net/ipmux/article/details/45038869
版权声明：本文为博主原创文章，转载请附上博文链接！
