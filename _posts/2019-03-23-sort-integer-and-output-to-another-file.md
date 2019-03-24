---
layout: post
title:  把一个文件中的整数排序后输出到另一个文件中
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```


#include <iostream>
#include <fstream>
#include <vector>
using namespace std;

template<class T>
bool decreOrder(T &a, T &b)
{
   return (a < b);
}
template<class T>
bool increOrder(T &a, T &b)
{
   return (a > b);
}

template<class T>
void Sort(vector<T> & v, bool (*compare)(T &,T &))
{
   int len = v.size();
   for (int i = 0; i < len-1; i++)
   {
       for (int j = len-1; j > i; j--)
       {
           if (compare(v[j-1],v[j]))
           {
               T temp;
               temp = v[j];
               v[j] = v[j-1];
               v[j-1] = temp;
           }
       }
   }
}

int main()
{
   vector<int> v;
   ifstream in("c:\\data.txt");
   if (!in)
   {
       cout << "Failed to open data.txt!" << endl;
       return 1;
   }

   while (!in.eof())
   {
       int temp;
       in >> temp;
       v.push_back(temp);
   }
   in.close();

   // sort the vector
   Sort<int>(v, increOrder<int>);

   // output the numbers in vector into file output.txt
   ofstream out("c:\\output.txt");
   if (!out)
   {
       cout << "Failed to open output.txt!" << endl;
       return 1;
   }

   for (int i = 0; i < v.size(); i++)
   {
       out << v[i] << endl;
   }

   out.close();
   cout << "End of the program" << endl;
   return 0;
}

```
