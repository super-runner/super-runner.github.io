---
layout: post
title:  markdown语法
date: 2019-03-20 16:30 +0800
categories: markdown
tags:  programming
author: Jason Chi
---
* content
{:toc}





### 标题

`# 一级标题`

`## 二级标题`

`### 三级标题`

共六级标题, 标准的语法是#后加一个空格

### 列表
##### 无序列表
- 无序列表可以这样写

  `- 无序列表可以这样写`
* 无序列表也可以这样写

  `* 无序列表也可以这样写`

##### 有序列表
有序列表则直接在文字前加 1.2.3. 符号要和文字之间加上一个字符的空格:
1. 有序列表1
2. 有序列表2

### 引用
>引用这样写

`>引用这样写`

### 图片与链接
插入链接与插入图片的语法很像，区别在一个 !号。插入图片的地址需要图床，生成URL地址即可。

##### 插入连接
[baidu](http://www.baidu.com)

`[baidu](http://www.baidu.com)`

##### 插入图片

![American National Park](https://i.loli.net/2019/03/21/5c9393955cb3b.jpg)

`![American National Park](https://i.loli.net/2019/03/21/5c9393955cb3b.jpg)`

### 粗体与斜体
**这是粗体写法**

`**这是粗体写法**`

*这是斜体写法*

`*这是斜体写法*`

### 表格
| 左对齐| 居中对齐|右对齐  |
| ------ |:--------:| -----:|
|Jason|37|Male|
|Becky|35|Female|

这个表格是这样写的:
```
| 左对齐| 居中对齐| 右对齐|
| ------ |:--------:| -----:|
|Jason|37|Male|
|Becky|35|Female|
```

### 代码框
`一行代码这样写`

\`一行代码这样写`

```
多行代码可以这样：
  3个`
   一段文字一段文字一段文字一段文字
   一段文字一段文字一段文字一段文字
   一段文字一段文字一段文字一段文字
  3个`
```

### 分割线
***

`***`

### 播放音乐
<audio controls="controls">
  <source src="/music/loveoncealife.mp3"  type="audio/mp3">
</audio>
