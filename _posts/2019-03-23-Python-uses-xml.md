---
layout: post
title:  python uses xml
date: 2019-03-23
categories: python
tags:  programming
author: Jason Chi
---
* content
{:toc}





```
#!/usr/bin/python

import xml.etree.ElementTree as etree

tree = etree.parse(wordbook.xml')
root = tree.getroot()
print len(root)

entries = tree.findall('item')

list = []
for word in entries:
 w = word.find('word')
 s =  w.text
        list.append(s)

f = file('words.txt', 'w')
CODEC = 'utf-8'
for item in list:
 f.write(item.encode(CODEC))
 f.write('\r\n')

f.close()
```
