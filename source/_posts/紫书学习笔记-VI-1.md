title: 紫书学习笔记_VI_1
tags:
  - zishu
categories:
  - acm
date: 2016-12-21 18:52:00
---
	我先把这个写在这……这两天每天都有东西一下子困惑我一大段的时间，前一阵是stl的set，昨天是关于md5为何不安全的问题，今天是这个破链表。
<!--more-->

---

是这样的，今天看到一个“链表”，与我之前所见到的全部不同，盯其小时无果，不得其逻辑，遂记于此，希望有思路之时能将其解开。
这个链表只用了一个数组当做指针，然后几乎就是动了动数组，就把链表建好了……我印象中链表是要结构体动态分配指针balabala，操作非常“复杂”（于我而言）
所以也有可能是我对链表的理解不够深，所以这种简易的写法也无法理解TAT

样例输入
This\_is\_a\_[Beiju]\_text
输出
BeijuThis\_is\_a\_text ;
即，“[”为home，“]”为end，对字符串进行键盘模拟。

```c++
#include <cstdio>
#include <cstring>
const int maxn=100000+5;
int last ,cur ,next[maxn];//光标位于标号cur号字符的前面
char s[maxn];

int main(){
  while(scanf("%s",s+1) == 1){
    int n = strlen(s+1);
    last = cur = 0;
    next[0] = 0;
    
    for(int i=1;i<=n;i++){
      char ch = s[i];
      if(ch=='[') cur = 0;
      else if(ch==']') cur = last;
      else{
        next[i] = next[cur];//最tm不明白的就在这个
        next[cur] = i;//和这个
        if(cur == last) last = i;//更新最后一个字符编号
        cur = i;//移动光标
      }
    }
    for(int i = next[0]; i != 0; i = next[i])
      printf("%c",s[i]);
     printf("\n");
  }
  return 0;
}
```

我大概写了半个整个程序工作的过程，略有所悟。
在没有[]的时候，就相当于模拟了一个数组下标，next[0]=1,next[1]=2...依次类推。


在有一个[号的时候，那一次的[的i下标的next是空的，然后其下一个值会继续指向下一个值，最后一个值会指向[1]，[0]会指向空下标的下一个值。
	输入abc[de
	next[0]=5
	next[1]=2
	next[2]=3
	next[3]=0
	next[4]无 
	next[5]=6
	next[6]=1
	整个顺序就是5-6-1-2-3，与输入是相应的(输入时下标为123456)
	注意0是个虚拟的光标
也就是说，整个下标指向的是s真正的下标。先从[之后的值开始，然后到开头，正如BeijuThis is一样。
以下胡言乱语

------
	还算比较明白的知道，next[cur]=i是指把next[cur]更新至最新处。由于i不会改变，改变的只会是cur的值，于是我们知道，cur就是个指针(于是我说了一通废话)，“光标的位置位于cur的右边”，即是说，当前光标位于s[cur]的右边，cur=0指向s[0]的右边，输入一个字符的话就是s[1]，具体的实现就是next[i]=next[cur]。
	cur变化至0，则next[i]指向光标，就是说此处的字符应该都去开始处了。之后cur继续向前，即从刚才向后的所有字符的次序全部被提前了。
	
	但是问题还是在于具体实现。
	next[i]=next[cur]，i的next=cur的next，就是说i的下一个指向当前光标的下一个，之后next[cur]=i，cur=i先把当前光标处的下一个指向最前，并把光标放到最前。

---------------------

_先记这些,嗯_