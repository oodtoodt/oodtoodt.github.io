---
title: dp2

date: 2018-03-27 20:20:35

tags:
- special

categories:
- acm

---
为了教我自己的不会的dp两天里做的...

<!--more-->

动态规划是通过**拆分问题，**定义问题状态和状态之间的关系，使得问题能够以递推（或者说分治）的方式去解决。


#### hdu1024

报警了啊....输出k个最大连续子序列的和最大的值。

果然我的dp水平还停留在最大连续子序列的阶段...

$dp[i][j]$ 前i个数，组成j组的和的最大值
决策：第i个数包含在第j个组里，或者自己单独成组。
$dp[i][j] = max(dp[i-1][j] + a[i] ,max(dp[0][j-1]～dp[i-1][j-1]) + a[i])$
其中$max(dp[0][j-1]～dp[i-1][j-1])$指的是前面i个数放在j-1个组里的最大值。为了使单独成组后和最大。
但是n和m都很大。所以还是要化成一维。$dp[i][j] 只与 dp[?][j]和dp[?][j-1]有关$，可以滚动数组。注意到遍历dp[i]时可以记录上面的max值。
输出时应为max值，而不是dp[n]。。。照搬题解还做错我也是(T ^ T)

不大行，基本上是照搬题解。感觉是好题。

核心代码

```c++
        for(int j = 1; j <= m; j++)
        {
            int maxd = -INF;
            for(int i = j; i <= n; i++)
            {
                dp[i] = max(dp[i-1]+a[i],maxdp[i-1] + a[i]);
                maxdp[i-1] = maxd;
                maxd = max(maxd,dp[i]);
            }
        }
```

#### hdu1029

给定一串数字，问出现次数大于某个值的数字是什么。
不用dp的话map可以水过。
dp的话，$dp[i][j]$表示...算了不dp了。

#### hdu1069
我是不是不适合做dp。。
典型dp。
题意：给出很多数目不限的方块的三维数据，然后问在长宽严格递减的情况下，如何摆放着使得高最大？
做法：枚举每一个方块的所有摆放方式，然后排序长度、宽度，对于长度、宽度做关于高度的lis...
回想一下正常的lis的话：
$𝑑𝑝[𝑘]=max(𝑑𝑝[𝑖]+1, 𝑑𝑝[𝑘]) $
```c++
for(int i = 0; i < cnt; i++)
	for(int j = i; j < cnt; j ++)
		if(a[j].x > a[i].x && a[j].y > a[i].y)
			dp[j] = max(dp[j],dp[i]+ve[j].z);
```
dp[i]表示扫到第i个元素最大高度，然后遍历所有元素，对于每个比第i个元素长宽大的位置更新dp为这个地方的dp或者是第i个元素+高。
硬要写只能写出这个来...真讲的话没法讲的。

#### hdu1074
题意：给出一些任务，给出任务的deadline和要做的话需要的天数，问如何安排使得所有任务超过deadline的天数总和最少。
想了半天不会写方程。
看了题解，是状压dp。
>对于完成123 和132来说，消耗的天数一定相同，只是完成顺序不同而扣的分不同，所以可以将完成x相同任务的状态压缩成一种状态并记录扣的最少分即可。
>即状态压缩dp。
>对于到达状态i，从何种状态到达i呢？只需要枚举所有的任务
>对于k，i中若k已完成，那么i就是由未完成k的状态到达完成了k的状态。

看到这一切就开朗起来了。所以就直接从小到大枚举所有状态，然后枚举到达状态的所有可能的状态，取超过deadline的天数总和的最小值作为dp值。

写一下代码。

记录下完成i的时间，另外注意每次更新dp、时间、还要更新输出顺序。

```c++
/*
  ID: oodt
  PROG:
  LANG:C++
*/
#include<iostream>
#include<algorithm>
#include<cstdio>
#include<cmath>
#include<string>
#include<cstring>
#include<map>
#include<vector>
#include<queue>
#include<stack>
#include<set>

using namespace std;

const int maxx=20;
int n,m,k;
int dead[maxx];
int cost[maxx];
string s[maxx];
int ans = 0,cnt = 0,pos = 0;
int l = 0,r = 0;
int dp[100000];
int t[100000];
int pre[100000];
const int INF = 0x3f3f3f3f;
vector<string> v;


int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/8-dp/简单(easy?)dp/hdu1074.txt","r",stdin);
#endif
    int T;
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d",&n);
        for(int i = 0; i < n; i++)
        {
            cin>>s[i]>>dead[i]>>cost[i];
        }
        int cnt = 1 << n;
        dp[0] = 0;
        memset(t,0,sizeof(t));
        for(int i = 1; i < cnt; i++)
        {
            dp[i] = INF;
            int pos = i;
            for(int j = 0; j < n; j++)
            {
                int temp = (1<<j);
                if(!(i & temp) || temp > i)continue;
                temp = i-temp;
                int te = t[temp] + cost[j] - dead[j];//j扣分数
                te = max(te,0);
                if(dp[temp] + te < dp[i])
                {
                    t[i] = t[temp] + cost[j];
                    dp[i] = dp[temp] + te;
                    pre[i] = j;
//                    printf("dp[%d](%d) = dp[%d](%d) + %d\n",i,dp[i],temp,dp[temp],te);
//                    printf("pre[%d] = %d\n",i,j);
                }
                if(dp[temp] + te == dp[i]){
                    if(s[j] > s[pre[i]]) {
                        pre[i] = j;
//                        cout<<s[j]<<s[pre[i]]<<endl;
//                        printf("pre[%d] = %d\n",i,j);
                    }
                }
            }
        }
        cnt--;
        printf("%d\n",dp[cnt]);
        v.clear();
        while(cnt)
        {
            v.push_back(s[pre[cnt]]);
            cnt = cnt - (1<<pre[cnt]);
        }
        for(int i = v.size()-1; i >= 0; i--)
        {
            cout<<v[i]<<endl;
        }
    }
    return 0;
}

```

#### hdu 1087

题意就是求子序列递增且和最大的值。

类似lis，其中dp的含义改为和的最大值。

#### hdu1257

题意求多少个递减序列能遍历所有的元素。可以贪心更新每个递减序列的最小值...更好的做法是求一个lis就行了。另外，最长上升子序列的长度就是不下降子序列的个数...

#### poj1458

lcs裸题…lcs重点在于用二维表示i和j。

####  hdu1114

题意：给出一个存钱罐的本体重和存满重，给出不同钱币样式的重量和价值，问使得存钱罐存满（一定满）的钱价值最小是多少。

完全背包，求使背包满的最小值。max改成min。

#### hdu1176

题意：天上掉锅。要尽量接住更多的锅，但是你很笨，每秒只能移动一个距离，给出锅下的坐标和时间，坐标一维，只有0-10这11个点。锅数100000。

看到n100000我以为凉了的，想到只有0-10，又想了下状态压缩，我丢人了。

$dp[i][j]$表示i时间在j点接住的锅数。用$a[i][j]$记录i时间j掉的锅数(很重要）。枚举所有时间i，所有点j，有

$dp[i][j] = max(dp[i-1][j],dp[i-1][j+1],dp[i-1][j-1]) + a[i][j];$

#### hdu1260

简单dp啦。。问两个人付和一个人付的所有情况里付钱最少的情况
$        dp[i] = min(dp[i-2]+b[i-1],dp[i-1]+a[i]);$

####  hdu1160
给出每个老鼠的w和s，求一个老鼠序列s.t. w严格递增，s严格递减
类似1069，只是变成+1

#### poj1015
做这个题把自己做进去了....
题意：给出n人的d和p值，求m个人的d值和sumd，p值和sump，abs(sumd-sump)最小的情况，如果有相同的输出sumd+sump最大的情况。d和p不超过20。n<200,m<20。

我一开始当成和上个题上不多的dp了。。写完了样例也过了，却一直wa。找了点数据，又回头看，自己的三重循环并不是对的——对于m个人的限制来说，并不是说更新值就一定是最优的，可能更到sumd远大于sump然后最后来一个反着差的数据使得其变成最优，而不是说每次最优结果一定最优。

明白了这一点后结合d和p不超过20就能发现这其实是背包吧...
讲道理为一开始都没注意到题目里藏了个条件没用orz。

$dp[i][j]$表示从n个人中选出差为j的和最大值。
发现j并不好求，那么就去掉绝对值，只求d-p。然后先行+400(20 * 20)使得不会出现负数并且能够最后求得正确的值。
说一下为什么要选这样的j。因为每次更新的时候对于每个确定的j，其前面的状态是确定的。即可以这样推上来最优。
贴代码。（好像还没写好。。

#### poj1221
拆成对称的且先递增后递减的整数拆分。
整数拆分 $dp[i][j]$是数i最大不超过j的划分数。
```c++
i < j  dp[i][j] = dp[i][i]
i == j dp[i][j] = dp[i][j-1] + 1;
i > j  dp[i][j] = dp[i][j-1]+dp[i-j][j];
```
这道题的话有两种情况——首先这样一个划分必定是abcba或者abba这两种情况，而后者只会出现在偶数中，而前面那个则要求n-c为偶数，这时候只需要看左边区间的划分就能得到所有情况，划分数就是$dp[(n-c)/2][c]$，