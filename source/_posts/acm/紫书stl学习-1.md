title: 紫书stl学习_1
tags:
- zishu
- stl

categories:

- acm

date: 2016-12-13 21:27:00
---
	初接触stl，感觉这玩意强的不行，但总感觉有点驳杂而繁复，少一点美感
---
<!--more-->

	这几个题都几乎是照着题解写的23333……不过很简单，思路也很简单，就是实现……
	stl的运用。

#### 5-1代码对齐 uva1593
##### 题意：给出若干行代码，要求各列单词左对齐且左边界对齐。

##### 思路：记录每一列的最大值，输出即可
(这个题似乎用数组+字符串也可以做，但我还是用了vector和string)
(但是字符串有个坏处，就是scanf比较傻。)
##### 代码
{% codeblock  lang:cpp  代码对齐 %}
	#include <iostream>
	#include <string>
	#include "sstream"
	#include "vector"
	#include "cstdio"
	
	using namespace std;
	
	int main()
	{
	    freopen("input.txt","r",stdin);
	    freopen("output.txt","w",stdout);
	    vector<string> vs[1005];
	    string word,line;
	    int maxx[1005]={};
	    int i=0,lie=0;
	    while(getline(cin,line))//这里用到了getline这个读取一行的函数
	    		//讲道理这个函数吼啊，有效搞定各种scanf玩不转的东西
	    {
	        stringstream ss(line);//把line重新转成流
	        while(ss>>word)//然后再读取。这整个一套大概 也许是一个体系233
	        {
	            if(maxx[lie]<word.size())
	                maxx[lie]=word.size();//取最大值
	            vs[i].push_back(word);//push_back,后添
	            lie++;//记录列值
	        }
	        i++;//记录行值
	        lie=0;
	    }
	    for(int j=0;j<i;j++)
	    {
	        for(int k=0;k<vs[j].size();k++)
	        {
	            cout<<vs[j][k];
	            int temp=maxx[k]-vs[j][k].size();
	
	            if(k==vs[j].size()-1)
	                cout<<endl;
	            else
	            {
	                for(int p=0;p<temp;p++)
	                {
	                    cout<<" ";//输出剩下的空格
	                }
	                cout<<" ";//输出列与列间空格 否则输出\n
	            }
	        }
	    }
	    return 0;
	}
{% endcodeblock %}

#### 5-2 Ducci序列uva1594
##### 题意
对于n元组$(a_1,a_2,...,a_n)$,可以得到新的组$(\\left| a_1-a_2 \\right|,\\left| a_2-a_3\\right| ,...,\\left|a_n-a_1\\right| )$。重复此过程，在保证1000步内可出结果的情况下，问是否变成$(0,……0)$或循环。

##### 思路
运用集合，可以简单的完成查找的工作从而使得判断循环变得简单。
不过set里貌似不能放数组，因为数组自身没有比较需要自己把运算符重载一遍……（与sort函数有些类似的意味）然而我写了一下午都没有成功……我对重载什么的太不熟悉了。
后来问了学姐，学姐告诉我，可以用类包住它再试……不过那样跟vector就没有什么太大的区别了……我只是想探究set里能放啥 为啥不能放数组。数组的地址不变，所以不会因为值的改变而改变（本质地址、指针）。底层群众就是吊啊……
不过如果用struct或者class包住的话也能有一些有趣的功用啊2333
不过研究一下午 发现c++tm的博大精深的有点离谱啊……不知真的学进去要到何种天昏地暗的程度。

##### 代码
{% codeblock  lang:c++  代码对齐 %}
	#include <iostream>
	#include <string>
	#include "sstream"
	#include "vector"
	#include "set"
	#include "queue"
	#include "cstdio"
	#include "cmath"
	#include "cstring"
	
	using namespace std;
	
	int main()
	{
	    freopen("input.txt","r",stdin);
	    freopen("output.txt","w",stdout);
	    int n,m;
	    scanf("%d",&m);
	    set <vector <int> > s;
	    for(int iii=0;iii<m;iii++)
	    {
	        vector<int> v;
	        s.clear();
	        scanf("%d",&n);
	        //memset(a,0,sizeof(a));
	        for(int jjj=0;jjj<n;jjj++)
	        {
	            int a;
	            cin>>a;
	            v.push_back(a);
	        }
	        //printf("%d",a[0]);
	        int t=0;
	        while(1)
	        {
	            if(!s.count(v))
	            {
	                s.insert(v);
	            }
	            else
	            {
	                printf("LOOP\n");
	                break;
	            }
	            int temp=v[0];
	            for(int i=0;i<n-1;i++)
	            {
	                v[i]=abs(v[i]-v[i+1]);
	            }
	            v[n-1]=abs(v[n-1]-temp);
	            t++;
	            int p=0;
	            for(int i=0;i<n;i++)
	            {
	                if(v[i]!=0)
	                {
	                    p=1;
	                    break;
	                }
	            }
	            if(p==0)
	            {
	                printf("ZERO\n");
	                break;
	            }
	            if(t==1000)
	            {
	                printf("LOOP\n");
	                break;
	            }
	        }
	    }
	    return 0;
	}
{% endcodeblock %}

#### 5-3 卡牌游戏
#####题意
有编号1-n的纸牌依次放在桌上，1在牌顶，每次扔掉第一张牌，然后把新的第一张牌放到整叠牌的最后。直到只剩1张。输出扔掉的牌和剩下的那张牌
#####思路
没有stl的模拟大概会很繁……
有了stl队列就行了……
输出front，pop,取出front，pop,push....循环即可。

##### 代码

```c++
#include "cstdio"
#include "cstring"
#include "algorithm"
#include "set"
#include "vector"
#include "map"
#include "queue"

using namespace std;

int main()
{
    freopen("input.txt","r",stdin);
    freopen("output.txt","w",stdout);
	int n,m;
	while(scanf("%d",&n)&&n)
	{
	    queue<int> q;
		for(int i=1;i<=n;i++)
		{
			q.push(i);
		}
		printf("Discarded cards:");
		while(q.size()>1)
		{
			int t=q.front();
			printf(" %d",t);
            q.pop();
            t=q.front();
            q.pop();
            q.push(t);
			if(q.size()!=1)
			printf(",");
		}
		printf("\nRemaining card: %d\n",q.front());
	}
	return 0;
}
```



#### 5-4 交换学生
##### 题意
对于给出的数对(a,b)，必须在n个数对中找到相应的数对(b,a)，其中(a,b)可能有很多个。判断给出的n个数对是否满足条件。
##### 思路
用map记录、搜索。
对于给出的(a,b),遍历map find(b)，查看其对应值是否为a，如果对应则用erase抹除map中的值，若无对应则加入map，最后检查map.empty即可。
注：因为可能的(a,b)有很多个,所以要用 multimap实现多个键的重复。大概就是，set是集合但是是用rbtree建的，map也类似，map本与set相似，不允许键与键的重复……
算了自己讲还是僵硬，引用一下
>multimap 与 map 一样，都是使用红黑树对记录型的元素数据,按元素键值的比较关系，进行快速的插入、删除和检索操作，所不同的是 multimap 允许将具有重复键值的元素插入容器。在 multimap 容器中，元素的键值与元素的映照数据的映照关系，是多对多的，因此，multimap 称为多重映照容器。multimap 与 map 之间的多重特性差异，类似于 multiset 与 set 的多重特性差异。



##### 代码
{% codeblock  lang:cpp  交换学生 %}
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
//    freopen("output.txt","w",stdout);
    int n;
    while(scanf("%d",&n)&&n)
    {
        multimap <int ,int > m;
        for(int i=0;i<n;i++)
        {
            bool flag=true;
            int a,b;
            scanf("%d%d",&a,&b);
            for(multimap<int,int > ::iterator i=m.find(b); i->first==b&&i!=m.end();i++)
            {
                if(i->second==a)
                {
                    m.erase(i);
                    flag=false;
                    break;
                }
            }
            if(flag==true)
            {
                m.insert(make_pair(a,b));
            }
        }
        if(m.empty())
            cout<<"YES"<<endl;
        else
            cout<<"NO"<<endl;
    }
    return 0;
}
{% endcodeblock %}

---

好了就记到这里吧……这一篇大概拖了一周才搞出来……题很简单 说到底是我怠惰了。
**说到底是我怠惰了。**
*……总有走远了的感觉 真是心碎的不止一点点。*