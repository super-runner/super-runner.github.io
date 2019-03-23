---
layout: post
title:  "容器map及pair的使用"
date: 2019-03-23
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#include <iostream>
#include <map>
using namespace std;

bool fncomp (char lhs, char rhs) {return lhs>rhs;}

struct classcomp
{
  bool operator() (const char& lhs, const char& rhs) const
  {
      return lhs<rhs;
  }
};

int main ()
{
    map<int,string,greater<int>> mapstring;  // compare= less or greater
    map<int,string,greater<int>>::iterator it;

    //map<int, string> mapstring;  // compare function is less() if not appoint
    //map<int,string>::iterator it;
    mapstring.insert(pair<int,string>(6, "Mongo"));
    mapstring.insert(pair<int,string>(3, "Anson"));
    mapstring.insert(make_pair(2, "Becky"));
    mapstring.insert(make_pair(5, "David"));
    mapstring[4] = "Jobs";
    mapstring[1] = "Jason";

    for (it=mapstring.begin(); it!=mapstring.end(); it++)
    {
        cout << (*it).first << " : " << (*it).second << endl;
    }//pair has 2 members: first->key and second->value

    cout << endl << "To delete 3 and 5, then replace 4:" << endl << endl;
    it = mapstring.find(3);
    mapstring.erase(it);     // erase by iterator
    mapstring.erase(5);      // erase by key
    mapstring[4] = "Titan";  // replace value by key

    for (it=mapstring.begin(); it!=mapstring.end(); it++)
    {
        cout << (*it).first << " : " << (*it).second << endl;
    }

    map<char,int> first;

    first['a']=10;
    first['b']=30;
    first['c']=50;
    first['d']=70;

    map<char,int> second (first.begin(),first.end());
    map<char,int> third (second);
    map<char,int,classcomp> fourth;                 // class as Compare
    bool(*fn_pt)(char,char) = fncomp;
    map<char,int,bool(*)(char,char)> fifth (fn_pt); // function pointer as Compare
    //map<char,int,bool(*)(char,char)> fifth (first.begin(),first.end(),fn_pt); // can initialize like this

    fifth.emplace('e', 20);
    fifth.emplace('f', 40);
    fifth.emplace('g', 60);
    fifth.emplace('h', 80);

    for (map<char,int>::iterator it=fifth.begin();it!=fifth.end();it++)
    {
        cout << (*it).first << " : " << (*it).second << endl;
    }

    /* demonstrate how pair works */
    pair<int,string> p1(3,"Great");
    pair<int,string> p2;
    p1 = make_pair(5,"Good");

    cout << p1.first << " : " << p1.second << endl;
    cout << p2.first << " : " << p2.second << endl;
    cout << (p1 > p2) << endl; // compare value, not key

    return 0;
}

```
