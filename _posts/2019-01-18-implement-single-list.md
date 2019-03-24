---
layout: post
title:  c++实现一个单链表
categories: c++
tags:  programming
author: Jason Chi
---
* content
{:toc}




```
#include <iostream>
#include <string.h>
#include <locale>
using namespace std;

template <typename T>
struct node{
    node * next;
    T data;
};

template <typename T>
using Node = node<T>;

template<typename T>
class myList
{
public:
    myList();
    ~myList();
    void add(T value);
    void showList(void);
    int getCount();
    //if class doesn't have count variable, this method works
    int length();
    Node<T> * find(int pos);
    Node<T> * insert(int pos, T value);
    virtual Node<T> * modify(int pos, T value);

private:
    Node<T> * head;
    Node<T> * tail;
    int count;
};

template<typename T>
myList<T>::myList(void)
{
    head = tail = NULL;
    count = 0;
}

template<typename T>
myList<T>::~myList()
{
    //cout << "before destruct, count = " << count << endl;
    Node<T> * p = head;
    while (head != NULL)
    {
        delete head;
        p = p->next;
        head = p;
    }

    tail = NULL;
    count = 0;
}

template<typename T>
void myList<T>::add(T value)
{
    if (head == NULL)
    {
        head = tail = new Node<T>;
        head->next = NULL;
        head->data = value;
    }
    else
    {
        tail->next = new Node<T>;
        tail = tail->next;
        tail->next = NULL;
        tail->data = value;
    }
    count++;
}

template<typename T>
void myList<T>::showList(void)
{
    Node<T> * p = head;
    while (p != NULL)
    {
        cout << p->data << endl;
        p = p->next;
    }  
}

template<typename T>
int myList<T>::getCount()
{
    return count;
}

template<typename T>
int myList<T>::length()
{
    int len = 0;
    Node<T> * p = head;
    while (p != NULL)
    {
        len++;
        p = p->next;
    }
    return len;
}

template<typename T>
Node<T> * myList<T>::find(int pos)
{
    if (pos < 0)
    {
        cout << "Invalid position value!" << endl;
        return NULL;            
    }

    if (head == NULL)
    {
        cout << "List is empty!" << endl;
        return NULL;    
    }


    Node<T> *p = head;
    while (pos > 0)
    {
        if (p->next == NULL)
        {
            cout << "out of scope!" << endl;
            return NULL;
        }

        --pos;
        p = p->next;
    }
    return p;
}

template <typename T>
Node<T> * myList<T>::insert(int pos, T value)
{
    Node<T> * p = find(pos);
    if (p)
    {
        Node<T> *q = new Node<T>;   
        q->next = p->next;
        q->data = value;
        p->next = q;
        return q;
    }
    else
    {
        return NULL;
    }
}

template <typename T>
Node<T> * myList<T>::modify(int pos, T value)
{
    Node<T> *p = find(pos);
    if (p)
    {
        p->data = value;
        return p;
    }
    else
    {
        return NULL;
    }
}



int main()
{
    myList<int> ml;
    ml.add(0);
    ml.add(1);
    ml.add(2);
    ml.add(3);
    ml.add(4);
    ml.modify(2, 12);
    ml.insert(3, 13);
    ml.showList();

    #define TOFIND (4)
    cout << "To find["<< (int)TOFIND <<"]: ";
    if (ml.find(TOFIND))
    {
        cout << ml.find(TOFIND)->data << endl;
    }

    return 0;
}
```
