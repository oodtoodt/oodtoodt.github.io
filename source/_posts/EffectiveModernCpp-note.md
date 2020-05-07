---
title: EffectiveModernCpp-note
date: 2020-05-07 15:31:23
tags:
---

真绝了，没想到我又回来了学c++了
<!--more-->

---
## item5：优先考虑auto而非显式
auto好处多多
+ 可以简化语法，减少重复量
+ 更适合存储闭包
+ 必须初始化
+ 避免依赖类型快捷方式的问题
```c++
std::vector<int> v;
unsigned sz = v.size();
auto sz = v.size();//better
```
因为实际上，size的返回值是`std::vector<int>::size_type`，在32-bit和64-bit的windows上并不一定是相同的。auto保证了不会有问题
+ auto保证避免很难意识到的类型不匹配的错误
```c++
std::unordered_map<std::sring,int> m;
for(const std::pair<std::string,int>&p : m) {}//wrong！
for(const auto & p : m) {}
```
因为unordered_map的key是一个常量，所以应当是`std::pair<const std::string,int>`，但这真的很难发现，也真的很难避免，除非你无所不知（指知道）。


## item6：当auto推导出非预期类型时应使用显式类型初始化
这个就很离谱，我用auto是为了什么？
那么问题的原因是什么呢，是一些代理类会占据auto使得一些令人困惑的事情发生，导致一些可能原先有的，指向某些东西的指针悬挂。
解决就是显式初始化。

## Item7：区别使用()和{}创建对象
```c++
Widget w1(10)
Widget w2();
```
我这样放在一起，你应当会觉得它们都是初始化，但其实不是，后者会变成函数声明。
大括号则没有这种问题。
大括号还能禁止窄变换，就是隐式自动变换，`int sum{1.0+2.0}`
大括号的问题是会被`std::initializer_list<>`劫持，认为整个大括号内的都是一套里的东西，这有点像auto去识别大括号。（当然，必须是`<>`中的实参可以转化的情况）
这里可以提一嘴：
```c++
std::vector<int> v1(10,20);
std::vector<int> v2{10,20};
```
前者是10个元素都是20的vector，后者是元素值分别为10和20的vector。
