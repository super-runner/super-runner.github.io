---
layout: post
title:  C++小程序：示例面向对象设计
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

class Shape
{
public:
    virtual void display(void) = 0;
    virtual void setValue(void) = 0;
};

class Circle: public Shape
{
public:
    Circle(int x=1):radius(x){}
    ~Circle(){}
    virtual void display(void)
    {
        cout << "square is ";
        cout << 3.14*radius*radius << endl;
    }
    virtual void setValue(void)
    {
        cout << "Enter radium:" << endl;
        cin >> radius;
    }
private:
    int radius;
};

class Rectangle: public Shape
{
public:
    Rectangle(int x=1, int y=1):longth(x),width(y){}
    ~Rectangle(){}
    virtual void display(void)
    {
        cout << "square is ";
        cout << longth*width << endl;
    }
    virtual void setValue(void)
    {
        cout << "Enter longth:" << endl;
        cin >> longth;
        cout << "Enter width:" << endl;
        cin >> width;
    }
private:
    int longth;
    int width;
};

class Square: public Shape
{
public:
    Square(int x=1):longth(x){}
    ~Square(){}
    virtual void display(void)
    {
        cout << "square is ";
        cout << longth*longth <<endl;
    }
    virtual void setValue(void)
    {
        cout << "Enter longth:" << endl;
        cin >> longth;
    }
private:
    int longth;
};

int main()
{
    Circle a;
    Rectangle b;
    Square c;
    Shape * p;
    int what;
    bool loop = true;

    while(loop)
    {
        cout << "-------------------------" << endl;
        cout << "input a integer:" << endl;
        cout << "1: Circle" << endl;
        cout << "2: Rectangle" << endl;  
        cout << "3: Square" << endl;
        cout << "4: Quit" << endl;

        cin >> what;
        switch(what)
        {
            case 1: p = &a;break;
            case 2: p = &b;break;
            case 3: p = &c;break;
            case 4: loop = false;continue;
            default:
                    cout << "Invalid option!" << endl;
                    continue;
        }
        p->setValue();
        p->display();
    }

    return 0;
}
 
```
