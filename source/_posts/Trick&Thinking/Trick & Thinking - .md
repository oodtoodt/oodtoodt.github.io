---
title: Trick & Thinking - 1
date: 2020-01-16 16:57:58
tags:
- c++
- trick

categories:
- trick

---

在学习的过程中接触到了一些...稀奇古怪的玩意，大概都是一些指导思想或者是小Trick，这里一并记录一下。
<!--more-->

---

# 前言
因为这些东西牵涉甚广，所以不要想着一下子就理解或者搞懂，这篇坑我会慢慢补，先盖个大概出来。
因为其实很多本质上是很高层的东西，所以可能也许以后会当作读物放松的时候看一下，但不是现在。

## CPS

## closures

## AST

## exception safe


## Interface As Contract
read《From Apprentice To Artisan》
https://stackoverflow.com/questions/219425/interface-contract-class-object


## SFINAE

## ADT


## 语法的一致性问题
源自轮子哥http://www.cppblog.com/vczh/archive/2013/04/27/199765.html
关于C语言的“定义和使用相一致”还有最后一个例子，这个例子也是很美妙的:
```c
int a;
typedef int a;

int (*f)(int a, int b);
typedef int (*f)(int a, int b);
```
typedef是这样的一个关键字：他把一个符号从变量给修改成了类型。所以每当你需要给一个类型名一个名字的时候，就先想一想，怎么定义一个这个类型的变量，写出来之后往前面加个typedef，事情就完成了。

不过说实话，就一致性来讲，C语言也就到此为止了。至于说为什么，因为上面这几条看起来很美好的“定义和使用相一致”的规则是不能组合的，譬如说看下面这一行代码：
```c
typedef int(__stdcall*f[10])(int(*a)(int, int));
```
这究竟是个什么东西呢，谁看得清楚呀！而且这也没办法用上面的方法来解释了。究其原因，就是C语言采用的这种“定义和使用相一致”的手法刚好是一种解方程的手法。譬如说`int *b`;定义了“`*b是int`”，那b是什么呢，我们看到了之后，都得想一想。人类的直觉是有话直说开门见山，所以如果我们知道`int*`是int的指针，那么`int* b`也就很清楚了——“b是int的指针”。 

因为C语言的这种做法违反了人类的直觉，所以这条本来很好的原则，采用了错误的方法来实现，结果就导致了“坑”的出现。因为大家都习惯“`int* a;`”，然后C语言告诉大家其实正确的做法是“`int *a;`”，那么当你接连的出现两三个变量的时候，问题就来了，你就掉坑里去了。
这个时候我们再回头看一看上面那一段长长的函数指针数组变量的声明，会发现其实在这种时候，C语言还是希望你把它看成“`int* b;`”的这种形式的：f是一个数组，数组返回了一个函数指针，函数返回int，函数的参数是`int(*a)(int, int)`所以他还是一个函数指针。

我们为什么会觉得C语言在这一个知识点上特别的难学，就是因为他同时混用了两种原则来设计语法。那你说好的设计是什么呢？让我们来看看一些其它的语言的作法：
```c++
C++:
function<int __stdcall(function<int(int, int)>)> f[10];

C#:
Func<Func<int, int, int>, int>[] f;

Haskell:
f :: [(int->int->int)->int]

Pascal:
var f : array[0..9] of function(a : function(x : integer; y : integer):integer):integer;
```