title: 12.2_vijos_约瑟夫环10e100
tags:
  - vijos
  - ''
  - Joseph
categories:
  - acm
date: 2016-12-02 17:56:00
---

去tm的约瑟夫环
<!--more-->
---


我就从来没想到过约瑟夫环能递推……大概我数学真心不敏感吧。
记得第一次接触约瑟夫环，我看着链表生无可恋。后来知道了可以用求余这种神奇的运算完成约瑟夫环……现在……我又知道了一种做法。
k=2时有显式表达式，k>2时有递推关系。
递推公式（m是第m个人去死，n是一开始人数，t是最后一个去死的）

$$
\\left\\{
\\begin{aligned}
&f[1]=0 \\\\
&f[i]=(f[i-1]+m)\\%i (i>1)
\\end{aligned}
\\right.
$$

显式表达式
k=2时， 取a使得
$$
2^a+1>n>2^a=m\\\\
t=(n-m)*2+1
$$
k=2时又由显式表达式可推得另一种神奇的方法：
把总数n化为2进制，将第一位的1移到最后一位……啧。

好了说正题
约瑟夫问题10E100版
k=2,100位n。(啧，世界哪来这么多人)
一开始没在意10e100，一个劲的莽RE……然后发现卧槽高精度，报警。果断找题解，于是有了以上内容。啧……不过怎么样都躲不过模拟的，再怎么搞100位的模拟都是很麻烦的事情，大概2进制的那个最简单了吧。

---

12.4
一做就是两天。
神经病啊！
最简单也要高精啊！
我还是怕高精……
真的怕高精……
说什么也怕高精……
{% codeblock  lang:cpp 约瑟夫环10e100版 %}
	#include "cstdio"
	#include "cmath"
	#include "cstring"
	#include "algorithm"
	#include "string"

	using namespace std;

	void change(char ch[],int a[],int n)
	{
	    for(int i=0;i<n;i++)
	    {
	     //   a[n-i-1]=ch[i]-'0';
	        a[i]=ch[i]-'0';
	    }
	}
	void change_2(int b[],int d[],int c[],int n)
	{
	    int t=0;
	    d[0]=1;
	    for(int k=0;k<n;k++)
	    {
	        if(b[k]==1)
	        {
	            for(int i=t;i<k;i++)
	            {
	                for(int j=0;j<400;j++)//to sicu the 2^n
	                {
	                    d[j]=2*d[j];
	                }
	                t=k;
	                for(int j=0; j<400; j++)//in wei
	                {
	                    if(d[j]>=10)
	                    {
	                        d[j]=d[j]-10;
	                        d[j+1]++;
	                    }
	                }
	            }
	            for(int j=0;j<400;j++)
	            {
	                c[j]+=d[j];
	                if(c[j]>=10)
	                {
	                    c[j]-=10;
	                    c[j+1]++;
	                }
	            }
	        }
	    }
	    int w;
	    for(int i=399;i>=0;i--)
	        if(c[i]!=0)
	        {
	            w=i;
	            break;
	        }
	        for(int i=w;i>=0;i--)
	            printf("%d",c[i]);
	}
	int solve(int a[],int b[],int n)
	{
	    int i,j,k,m;
	    int beichushu,shang,yushu;
	    int temp[400];
	    memset(temp,0,sizeof(temp));

	    j=0;
	    for(;;)
	    {
	        int p=0;
	        for(i=0;i<n;i++)
	        {
	            beichushu=a[i];
	            if(beichushu==0)
	            {
	                a[i]=0;
	                yushu=0;
	            }
	            else
	            {
	                p=1;
	                shang=beichushu>>1;
	                a[i]=shang;
	                yushu=beichushu-(shang<<1);
	            }
	            if(i!=n-1)
	            {
	                a[i+1]+=yushu*10;
	            }
	            else
	            {
	                if(p==0)
	                    return j;
	                j++;
	                b[j-1]=yushu;
	            }
	        }
	    }

	}

	int main()
	{
	  //  freopen("input.txt","r",stdin);
	    int n,m,i;
	    char ch[100];
	    int a[400];//to save 10 of ch;
	    int b[400];//to save 2 of ch;
	    int d[400];//to save 2^n;
	    int c[400];//to sum 2^n*a[i]
	    memset(a,0,sizeof(a));
	    memset(b,0,sizeof(b));
	    memset(c,0,sizeof(c));
	    memset(d,0,sizeof(d));
	    scanf("%s",ch);
	    n=strlen(ch);
	    change(ch,a,n);
	    int n1=solve(a,b,n);
	 //   int n1=strlen(b);
	    for(i=n1-1;i>=0;i--)
	    {
	     //   printf("%d ",b[i]);
	        b[i+1]=b[i];
	    }
	    b[0]=1;
	  //  for(i=0;i<n1;i++)
	    {
	  //      printf("%d",b[i]);
	    }
	    change_2(b,d,c,n1);
	    return 0;
	}
{% endcodeblock %}

---

ok，*每天心碎一点点。*	