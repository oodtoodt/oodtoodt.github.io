---
title: dp1

date: 2018-03-09 10:54:35

tags:
- special

categories:
- acm

---

dp专题。。。
dp把一个大佬同学劝退了....我好怕啊。。从刚进大学的时候看的题就是dp，当时感觉就是天书，用了两个小时去推到底在算啥。
感觉要步后尘了啊。
<!--more-->

---

###整数划分
1. 将n划分成m个正整数之和的划分数。（正好为m）
   `dp[i][j]`  表示i的划分中，j个数的分法
```c++
i<j:dp[i][j] = 0
i=j:dp[i][j] = 1
i>j:dp[i][j] = dp[i-1][j-1]+dp[i-j][j];
```
   包括1的情况 + 不包括1的情况（每个数-1）
   另外，将n划分成不大于m的划分数：
```c++
i>=j:dp[i][j] = dp[i][j-1] + dp[i-j][j];
i<j :dp[i][j] = dp[i][j-1]
```
   这里可以理解为有无0的情况，也可以理解为有无最大j的情况。

2. 将n划分成最大数不超过k的划分数。

`dp[i][j] `表示整数 i 的划分中，每个数不大于 j 的划分数。
```c++
i < j  dp[i][j] = dp[i][i]
i == j dp[i][j] = dp[i][j-1] + 1;
i > j  dp[i][j] = dp[i][j-1]+dp[i-j][j];
```
将n划分成最大数不超过k的划分数。不同的数
```c++
i < j  dp[i][j] = dp[i][i]
i == j dp[i][j] = dp[i][j-1] + 1;
i > j  dp[i][j]= dp[i][j-1]+ dp[i-j][j-1] 
```
跟上面那个一比，有没有什么发现？
3. 将n划分成若干奇正整数之和的划分数。
```c++
g[i][j]:将i划分为j个偶数
f[i][j]:将i划分为j个奇数

g[i][j] = f[i - j][j]; 
f[i][j] = f[i - 1][j - 1] + g[i - j][j];
```
>i中拿出j个1分到每一份中，将剩余的i-j分成j个奇数
>一份包含奇数1，剩余的i-1分成j-1个奇数；另一种，每份至少大于1，将j个1拿出来分到每一份中，其余i-j分成j份

##背包
### 01背包
超重要
```c++
f[i][v]=max{f[i-1][v],f[i-1][v-c[i]]+w[i]}
```
一维
```c++
for i=1..N
    for v=V..0
        f[v]=max{f[v],f[v-c[i]]+w[i]};
```
### 完全背包

裸思路

```c++
f[i][v]=max{f[i-1][v-k*c[i]]+k*w[i] | 0<=k*c[i]<=v}
```

一维且$O(VN)$
```c++
for i=1..N
    for v=0..V
        f[v]=max{f[v],f[v-cost]+weight}

f[i][v]=max{f[i-1][v],f[i][v-c[i]]+w[i]}
```
### 多重背包
$O(V*Σn[i])$
```c++
f[i][v]=max{f[i-1][v-k*c[i]]+k*w[i]|0<=k<=n[i]}
```
>多重背包问题同样有O(VN)的算法。这个算法基于基本算法的状态转移方程，但应用单调队列的方法使每个状态的值可以以均摊O(1)的时间求解。

#### cf922e
题意：n棵树，召唤第i棵树的鸟需要cost_i的能量，第i棵树有c_i个鸟，召唤一只鸟增加B的能量上限，初始有W的能量，W的上限，前进一棵树回X点能量，只能向前走，问最多召几只鸟。。。。。 n < 10^3 $\sum c_i <10^4$

用$dp[i][j]$表示走到第i课树，第j个鸟，剩下能量最多的情况。
转移方程：$dp[i][j] = max(dp[i-1][j],dp[i-1][j-k] - k*cost[i])$

三重循环，复杂度$n * \sum c_i $，题意有很明显的提示。。。

问题在这：如何获取能量的情况？这就跟dp数组挂钩了，使i加的时候+x，j加的时候上限+b吗？

最后抄了别人的代码：请细细体会能量的获取。

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

const int maxx=10005;
int n,m,k;
int a[maxx];
int cost[maxx];
int c[maxx];
int dp[1005][maxx];
int ans = 0,cnt = 0,pos = 0;
int l = 0,r = 0;
int W,B,X;

int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/8-dp/简单(easy?)dp/cf922e.txt","r",stdin);
#endif
    scanf("%d%d%d%d",&n,&W,&B,&X);
    int w = W;
    for(int i = 1; i <= n; i++)
    {
        scanf("%d",&c[i]);
    }
    for(int i = 1; i <= n; i++)
    {
        scanf("%d",&cost[i]);
    }
    memset(dp,-1,sizeof(dp));
    dp[0][0] = w;
    int tot = 0;
    for(int i = 1; i <= n; i++)
    {
        tot += c[i];
        int up = w;
        for(int j = 0; j <= tot; j++)
        {
            for(int k = 0; k <= j && k <= c[i]; k++)
            {
                if(dp[i-1][j-k] < k*cost[i]) continue;
                int t = min(up,dp[i-1][j-k]+X);
                dp[i][j] = max(dp[i][j],dp[i-1][j-k] - k*cost[i]);
            }
            if(dp[i][j] != -1) dp[i][j] = min(dp[i][j] + X,up);
            up += B;
        }
    }
    ans = tot;
    for(int i = 0; i <= tot; i++)
    {
        if(dp[n][i] == -1) ans --;
    }
    printf("%d\n",ans);
    return 0;
}

```


