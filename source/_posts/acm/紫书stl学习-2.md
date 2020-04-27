title: 紫书stl学习_2
tags:
- zishu
- stl

categories:

- acm

date: 2016-12-19 23:00:00
---
	这次题肯定不是上次那种动动脚趾头就能做的啦
<!--more-->

---

#### 5-5复合词 uva10391
##### 题意
给出一个字典，找出所有复合词，即恰好由字典中两个单词拼接而成的单词。输入已按字典序由小到大排好，且不超过120000个。
##### 思路
直接双循环肯定会炸，所以需要一些二分查找之类的东西。
毕竟字典所以无需重复，set查找。
核心学习： substr,set.find()迭代器
##### 代码

```c++
#include <iostream>
#include <string>
#include "sstream"
#include "vector"
#include "set"
#include "map"
#include "queue"
#include "cstdio"
#include "cmath"
#include "cstring"

using namespace std;

int main()
{
    freopen("input.txt","r",stdin);
    set <string > s;
    string str;
    while(cin>>str)
    {
        s.insert(str);
    }
    for(set<string > ::iterator it=s.begin();it!=s.end();it++)
    {
        string a=*it;
        for(int i=1;i<a.length();i++)
        {
            if(s.find(a.substr(0,i))!=s.end()&&s.find(a.substr(i,a.length()-i))!=s.end())
            {
                cout<<a<<endl;
                break;
            }
        }
    }
    return 0;
}

```

中间那段substr可以膜(学习)一波。

---

#### 5-6对称轴 uva1595
##### 题意
给出平面上一些点，问是否可以找到一条垂直于x轴的直线使所有点左右对称。N<=1000
##### 思路
首先是存点，然后操作，无序情况下不可能对每一次都套三层循环(乃至更多)。
吐槽……
神tm水的数据啊，居然直接求出y中所有x的平均值就能过。。在点的分布非常不均匀的时候，比如左边与轴偏离极大而右边偏离小但点多的情况下根本是不对的啊……
起初想用map的，毕竟每个x对应y值，但是想了想，如果只是把map当做数组用的话效率低不说还很难用，于是想到每次读取时直接判定……然后发现一开始不知道轴的位置，所以没法判定，这就比较僵硬……
中途用了一次set，用着用着想法跑偏了去求平均值了233，交上去ac之后感觉似乎不对，mdzz

最后还是用的vector，毕竟比起数组有一定的优势(在数据范围不大的情况下每次读取添加是可以用vector的但是我还是觉得每次读取添加太蠢了，效率爆炸)
vector直接sort，然后两边直接相加向中间靠拢，嗯。说到底是简化问题。
##### 代码

```c++
#include <iostream>
#include <string>
#include "sstream"
#include "vector"
#include "set"
#include "map"
#include "queue"
#include "cstdio"
#include "cmath"
#include "cstring"
#include "algorithm"

using namespace std;

//typedef set<int >Set;
map<int ,int >IDcache;
vector<int >Setcache;

int ID (int x)
{
    if(IDcache.count(x))return IDcache[x];
    Setcache.push_back(x);
    return IDcache[x]=Setcache.size()-1;
}
struct cyw{
    int x;
    int y;
};//q[1000];
int cmp1(cyw a,cyw b)
{
    return a.y<b.y;
}
int cmp2(cyw a,cyw b)
{
    if(a.y==b.y)
    return a.x<b.x;
}


int main()
{
    vector <int > v[1000];
    freopen("input.txt","r",stdin);
 //   freopen("output.txt","w",stdout);
    int p,n;
    scanf("%d",&n);
 //   map<int ,int > m;
    for(int ii=0;ii<n;ii++)
    {
        for(int i=0;i<1000;i++)
        {
            v[i].clear();
        }
        IDcache.clear();
        Setcache.clear();
        scanf("%d",&p);
   //     memset(q,0,sizeof(q));
        for(int i=0;i<p;i++)
        {
            int a,b;
            scanf("%d%d",&a,&b);
            int x=ID(b);
            v[x].push_back(a);
        }
        double t=0;
        int book=0;
        sort(v[0].begin(),v[0].end());
        t=(v[0][0]+v[0][v[0].size()-1])*1.0/2;
        for(int i=0;i<Setcache.size()&&!book;i++)
        {
            int temp=v[i].size();
            sort(v[i].begin(),v[i].end());
            for(int j=0;j<temp;j++)
            {
                double tt=(v[i][j]+v[i][temp-1-j])*1.0/2;
      //          printf("%.1f ",tt);
                if(tt!=t)
                {
                    book=1;
                    break;
                }
            }
        }
        if(book==0)
        {
           printf("YES\n");
        }
        else
        {
            printf("NO\n");
        }
    }
    return 0;
}
```
可以看到一些糟糕的遗留痕迹233

---


*近来懈怠不已，恐怕是期末临近之故*