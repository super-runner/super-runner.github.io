---
layout: post
title:  "[转]:C++在类的成员函数中，允许直接访问该类的成员对象的私有成员变量"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





[转自此处](https://blog.csdn.net/geophyboy/article/details/14119775)

问题（知识点）描述：
a． 在C++的类的成员函数中，允许直接访问该类的对象的私有成员变量。
b． 在类的成员函数中可以访问同类型实例的私有变量。
c． 拷贝构造函数里，可以直接访问另外一个同类对象（引用）的私有成员。
d． 类的成员函数可以直接访问作为其参数的同类型对象的私有成员。
举例描述上述知识：
1).在拷贝构造函数中可以访问引用对象的私有变量：例如:

```
class   Point
{
public:
       Point(int xx=0,int yy=0){X=xx;Y=yy;}
       Point(Point &p);
private:
       int   X,Y;
};
Point::Point (Point   &p)
{
       X=p.X;
       Y=p.Y;
}
```

2). 在类的成员函数中可以访问同类型实例的私有变量
```
class  A
{
public:
int getA() const { return a; }
  void setA(int val) { a = val; }
  void assign(A& _AA)
 {
  this->a = _AA.a;
  _ AA.a = 10; //可以直接访问
 }
   void display()
      {
        cout<<"a:"<<a<<endl;
      }
private:
   int a;
};
```
3).

```
#include <iostream>
using namespace std;
class TestClass
{
public:
    TestClass(int amount)
    {
        this->_amount = amount;
    }
    void UsePrivateMember()
    {
        cout<<"amount:"<<this->_amount<<endl;
        /*----------------------*/
        TestClass objTc(10);
        objTc._amount = 15;   //访问同类型实例的私有变量
        cout<<objTc._amount<<endl;
        /*----------------------*/
    }
private:
    int _amount;
};

int main()
{
    TestClass tc(5);
    tc.UsePrivateMember();
    return(0);
}
```
关于该问题（知识点）的讨论和或解释：
1. 私有是为了实现“对外”的信息隐藏，或者说保护，在类自己内部，有必要禁止私有变量的直接访问吗？难道还要让我们声明一个类
为自身的友元？
Point::Point(Point   &p)
{
        X=p.X;   //这在Point类中的Point函数内，所以没错。
        Y=p.Y;
}
请记住你是在定义你的类,不是在用。
2. C++的访问修饰符的作用是以类为单位，而不是以对象为单位。
通俗的讲，同类的对象间可以“互相访问”对方的数据成员，只不过访问途径不是直接访问.
步骤是：通过一个对象调用其public成员函数，此成员函数可以访问到自己的或者同类其他对象的public/private/protected数
据成员和成员函数(类的所有对象共用)，而且还需要指明是哪个对象的数据成员(调用函数的对象自己的成员不用指明，因为有this指
针；其他对象的数据成员可以通过引用或指针间接指明)
可以提供访问行为的主语为“函数”。
类体内的访问没有访问限制一说，即private函数可以访问public/protected/private成员函数或数据成员，同理，protected
函数，public函数也可以任意访问该类体中定义的成员public继承下，基类中的public和protected成员继承为该子类的public和
protected成员（成员函数或数据成员），然后访问仍然按类内的无限制访问。
3. 注意公有和私有的概念：每个类的对象都有自己的存贮空间，用于存储内部变量和类成员；但同一个类的所有对象共享一组类方法，
即每种方法只有一个源本。很明显，类方法是所有该类对象共同使用的，因此不存在每个对象有一组类方法的副本。源本的类方法自然
可以访问所有该类对象的私有成员。
4. 访问权限是相对于类而言，而非对象！
一个问题，为什么成员函数可以访问私有成员呢？结果很显然，如果不能访问，那么私有成员的存在就没有了意义。而对于成员函数中
允许访问对象的数据成员，一方面保证了安全性与封装性，另一方面提供方便的操作。第一句话的解释，就是承认只有成员函数可以访
问私有成员，这里不涉及友元及派生。这样一来，安全性仍然得到了保证，也完成了封装工作。对于第二句话，试想，如果都得靠接口
来实现数据传送，那么操作是否极为不便？既然处于成员函数中，已经保证了足够的安全和封装性，那么这里如果还得借助接口，就有
些不合情合理了。作为对数据成员的灵活处理，设计者允许在成员函数中访问对象的私有成员，为使用者提供了很大的方便。这同时也
反映了语言的灵活性和原则性.




C++中类访问权限控制：
第一：private, public, protected 访问标号的访问范围，在没有继承的情况下：
private：
只能由1.该类中的函数、2.其友元函数访问。
不能被任何其他访问，该类的对象也不能访问。
protected：
可以被1.该类中的函数、2.子类的函数、以及3.其友元函数访问。
但不能被该类的对象访问。
public：
可以被1.该类中的函数、2.子类的函数、3.其友元函数访问，也可以由4.该类的对象访问。
 注：友元函数包括3种：设为友元的普通的非成员函数；设为友元的其他类的成员函数；设为友元类中的所有成员函数。
第二：类的继承后方法属性变化，具体情况如下：
private 属性是不能够被继承的，protected继承和private继承能降低访问权限。
使用private继承，父类的protected和public属性在子类中变为private；
使用protected继承，父类的protected和public属性在子类中变为protected；
使用public继承，父类中的protected和public属性不发生改变;
 如下所示：
                   public:        protected:       private:
public继承          public        protected        不可用
protected继承       protected     protected        不可用
private继承         private       private          不可用

```
class A
{
 public:
     A(int i) : a(i){}
 protected:
     int a;
 };
class B : public A
{
 public:
     B(int i) : A(i) {}
     void fb(A  &a) {cout<<a.a<<endl;}
 };
 ```

 编译出错；若将倒数第二行改为
 void fb(B  &a) {cout<<a.a<<endl;}
 则编译运行都没问题：
 保护类型只能被本类中的方法或者派生类访问，不能被本类的具体对象访问。a是类A的保护类型成员，能被A中的方法（函数）访
 问，能被B类访问，不能被类A的具体对象A a访问。
 一个类的保护成员或私有成员只能被该类的成员函数或该类的友元函数来访问，外部代码不能访问，B类中的函数fb对于A类中的
 保护成员变量a来说是外部代码不能通过该类的对象直接访问，由于B类公有继承了A类，a在A类中是保护成员，则在B类中也是保
 护成员，因此B类中的函数可以访问自己的保护成员。
