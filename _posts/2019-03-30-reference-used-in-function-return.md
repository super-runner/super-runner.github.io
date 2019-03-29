---
layout: post
title:  引用在函数返回值中的使用
date: 2019-03-30
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}






#include <iostream>
using namespace std;

int array[10] = {0};
int error = -1;

int & put (int n)
{
    if (n<10 && n>=0)
    {
        return array[n];
    }
    else
    {
        cout <<"Input Error" << endl;
        return error;
    }
}


int main()
{
    int num, value;

    cout << "Please enter number:" << endl;
    cin >> num;
    
    while (num >= 0)
    {

        cout << "Please enter value:" << endl;
        cin >> value;
        put(num) = value;
        
        cout << "Please enter number:" << endl;
        cin >> num;
    }
    
    for (int i=0; i<10; i++)
    {
        cout << i << ": " << put(i) << ",  ";
    }
    
    cout << endl;
    cout << "Program end!" << endl;


    return 0;
}
