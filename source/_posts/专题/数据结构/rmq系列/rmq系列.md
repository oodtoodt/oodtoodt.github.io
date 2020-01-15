---
title:  rmq系列

date: 2018-07-15 21:33:34

tags:

- special
- xds

categories:
- acm

---

rmq系列。有一些之前写的题。多是些半途而废（对说的就是主席树和树状数组）的专题，有时间另专开几个讨论他们好了

<!--more-->

---

Rmq

就是区间查询的问题，有三（四？）种写法：线段树，树状数组（只能求和【存疑】），st（只能求最值【存疑】），（差分？）

写了几个线段树之后发现最值的话还是直接用st算了

------

### 划分树

> 主要用于快速求出(在log(n)的时间复杂度内）序列区间的第k大值。

划分树是从上向下，按排序依次把小于区间中位数的放在左子树，大于的放到右子树，维护每一层到i点进入左子树的数量，查询时直接获取是左还是右，是第几个，递归（思想其实像二分）。建树复杂度在nlogn，查询就是logn了。

其实也可以理解成一个线段树，先归并排序，存每一段的起点和终点，然后二分枚举查找区间内比当前枚举数小和相等的数是多少。其实大体思路相似。

板子（带了求和，是hdu3473的板子）：

```c++
const int MAXN = 100010;
ll suml[20][MAXN],sum[MAXN];//注意suml是单层，不要更新错
int tree[20][MAXN];//表示每层每个位置的值
int sorted[MAXN];//已经排序好的数
int toleft[20][MAXN];//toleft[p][i]表示第i层从1到i有数分入左边
void build(int l,int r,int dep)
{
    if(l == r)return;
    int mid = (l+r)>>1;
    int same = mid - l + 1;//表示等于中间值而且被分入左边的个数
    for(int i = l; i <= r; i++) //注意是l,不是one
    if(tree[dep][i] < sorted[mid]) same--;
    int lpos = l;
    int rpos = mid+1;
    for(int i = l;i <= r;i++) {
        if(tree[dep][i] < sorted[mid]) {
            tree[dep+1][lpos++] = tree[dep][i];
            suml[dep][i] = suml[dep][i-1] + tree[dep][i];//左进一
        }
        else if(tree[dep][i] == sorted[mid] && same > 0) {
            tree[dep+1][lpos++] = tree[dep][i];
            suml[dep][i] = suml[dep][i-1] + tree[dep][i];//左进一
            same--;
        }
        else {
            tree[dep+1][rpos++] = tree[dep][i];
            suml[dep][i] = suml[dep][i-1];
        }
        toleft[dep][i] = toleft[dep][l-1] + lpos - l;
    }
    build(l,mid,dep+1);
    build(mid+1,r,dep+1);
}
int lnum,rnum;
ll lsum;
//查询区间第k大的数,[L,R]是大区间，[l,r]是要查询的小区间
int query(int L,int R,int l,int r,int dep,int k) {
    if(l == r) {
        return tree[dep][l];
    }
    int mid = (L+R)>>1;
    int cnt = toleft[dep][r] - toleft[dep][l-1];
    if(cnt >= k)
    {
        int newl = L + toleft[dep][l-1] - toleft[dep][L-1];
        int newr = newl + cnt - 1;
        return query(L,mid,newl,newr,dep+1,k);
    }
    else
    {
        int newr = r + toleft[dep][R] - toleft[dep][r];
        int newl = newr - (r-l-cnt);
        lnum += cnt;
        lsum += suml[dep][r] - suml[dep][l-1];
        return query(mid+1,R,newl,newr,dep+1,k-cnt);
    }
}
```

#### hdu2665 poj2104

模板题

#### hdu3473

在一段区间内选一个数使得区间内其他数与它的差的绝对值和最小，并求这个和。关键：许多查询，区间，和。
上面的板子再加上

```c++
int main(){
    int T;
    scanf("%d",&T);
    while(T--)
    {
        printf("Case #%d:\n",++pos);
        scanf("%d",&n);
        for(int i = 1; i <= n; i++)
        {
            scanf("%d",&tree[0][i]);
            sum[i] = sum[i-1] + tree[0][i];
            sorted[i] = tree[0][i];
        }
        sort(sorted+1,sorted+n+1);
        build(1,n,0);
        scanf("%d",&m);
        for(int i = 1; i <= m; i++)
        {
            scanf("%d%d",&l,&r);
            l++,r++;
            lnum = 0,lsum = 0;
//            int p = ((l+r)>>1) + 1;
            int t = query(1,n,l,r,0,(r-l)/2+1);
            int rnum = r - l + 1 - lnum;
            ll rsum = sum[r] - sum[l-1] - lsum;
//            ll ans = rsum-lsum + t*(lnum-rnum);
            ll ans = t*lnum - lsum - (t * rnum - rsum);
            printf("%lld\n",ans);
        }
        //if(T != 0)
        cout<<endl;
    }
    return 0;
}
```

#### hdu4417

问如果马里奥最多能跳H高，那么能达到[L,R]里多少个方块？

二分+划分树....二分个数，划分树求出值与H比较大小即可。因为二分一直跑了个0的下标，一直在re。

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
int ans = 0,cnt = 0,pos = 0;
typedef long long ll;
int l = 0,r = 0;


const int MAXN = 100010;
ll suml[20][MAXN],sum[MAXN];//注意suml是单层，不要更新错
int tree[20][MAXN];//表示每层每个位置的值
int sorted[MAXN];//已经排序好的数
int toleft[20][MAXN];//toleft[p][i]表示第i层从1到i有数分入左边
void build(int l,int r,int dep)
{
    if(l == r)return;
    int mid = (l+r)>>1;
    int same = mid - l + 1;//表示等于中间值而且被分入左边的个数
    for(int i = l; i <= r; i++) //注意是l,不是one
    if(tree[dep][i] < sorted[mid]) same--;
    int lpos = l;
    int rpos = mid+1;
    for(int i = l;i <= r;i++) {
        if(tree[dep][i] < sorted[mid]) {
            tree[dep+1][lpos++] = tree[dep][i];
            suml[dep][i] = suml[dep][i-1] + tree[dep][i];//左进一
        }
        else if(tree[dep][i] == sorted[mid] && same > 0) {
            tree[dep+1][lpos++] = tree[dep][i];
            suml[dep][i] = suml[dep][i-1] + tree[dep][i];//左进一
            same--;
        }
        else {
            tree[dep+1][rpos++] = tree[dep][i];
            suml[dep][i] = suml[dep][i-1];
        }
        toleft[dep][i] = toleft[dep][l-1] + lpos - l;
    }
    build(l,mid,dep+1);
    build(mid+1,r,dep+1);
}
int lnum,rnum;
ll lsum;
//查询区间第k大的数,[L,R]是大区间，[l,r]是要查询的小区间
int query(int L,int R,int l,int r,int dep,int k) {
    if(l == r) {
        return tree[dep][l];
    }
    int mid = (L+R)>>1;
    int cnt = toleft[dep][r] - toleft[dep][l-1];
    if(cnt >= k)
    {
        int newl = L + toleft[dep][l-1] - toleft[dep][L-1];
        int newr = newl + cnt - 1;
        return query(L,mid,newl,newr,dep+1,k);
    }
    else
    {
        int newr = r + toleft[dep][R] - toleft[dep][r];
        int newl = newr - (r-l-cnt);
        lnum += cnt;
        lsum += suml[dep][r] - suml[dep][l-1];
        return query(mid+1,R,newl,newr,dep+1,k-cnt);
    }
}


int ok(int mid)
{
//    if()
    int t = query(1,n,l,r,0,mid);
    if(t > k) return 0;
    else return 1;
}

int erfen()
{
    int mid;
    int ll = 1,rr = r-l+1;
    while(ll<=rr)
    {
        mid = (ll + rr) / 2;
        //      printf("%d\n",mid);
        if(ok(mid))
        {
            ll = mid + 1;
        }
        else
        {
            rr = mid - 1;
        }
    }
//    ll--;
    return ll;
}


int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/5-数据结构/hfs划分树/hdu4417.txt","r",stdin);
#endif
    int T;
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d%d",&n,&m);
        for(int i = 1; i <= n; i++)
        {
            scanf("%d",&tree[0][i]);
            sum[i] = sum[i-1] + tree[0][i];
            sorted[i] = tree[0][i];
        }
        sort(sorted+1,sorted+n+1);
        build(1,n,0);
        printf("Case %d:\n",++cnt);
        for(int i = 0; i < m; i++)
        {
            scanf("%d%d%d",&l,&r,&k);
            l++,r++;
            int t = erfen();
            if(t > 0) t--;
            printf("%d\n",t);
        }
    }
    return 0;
}

```



##### st板子

```c++
int dp[maxx][20];
int mm[maxx];

void initrmq(int n,int b[])
{
    mm[0] = -1;
    for(int i = 1; i <= n; i++)
    {
        mm[i] = ((i&(i-1)) == 0)?mm[i-1]+1:mm[i-1];
        dp[i][0] = b[i];
    }
    for(int j = 1; j <= mm[n]; j++){
        for(int i = 1; i + (1<<j) -1 <= n; i++){
            dp[i][j] = max(dp[i][j-1],dp[i+(1<<(j-1))][j-1]);
        }
    }
}
int rmq(int x,int y)
{
    int k = mm[y-x+1];
    return max(dp[x][k],dp[y-(1<<k)+1][k]);
}
```

#### bzoj 1067

这个题太暴力了....我选择死亡。

认真点说的话，去网上查一下吧还是。大概要考虑6种情况，还要放到线段树里，st的话，还是算了吧。

#### hdu3486

题意：给出一串数(n<200000)，问最小选出多少个区间（区间长度=n / 区间个数）使得每个区间内的最大值的和大于给定的一个值？

一种做法是二分区间长度（个数），rmq查询最大值。但是有一个问题：二分不一定得到的是最优解，虽然过了但是我觉得很奇怪.....比如 1 1 1000 1000 1 1，大于1500的话，答案应该是2，但是二分时第一次选3个区间，过不了，就会往大里选，最后选到6。

嘛。。枚举会好一点。应该说会是正确的。应该是数据水了。

#### hdu3193

给出一些旅馆的di和pi，如果存在di和pi同时小于该旅馆的某个旅馆则放弃该旅馆，问最后剩下哪些旅馆？n < 10000

首先对其中一个数据从小到大排序，另一个数据次要从大到小排序，然后对次要数据做rmq，找到每个旅馆是不是最小值，如果是就扔进答案里。

如果不用rmq的话，感觉排序之后遍历记录值的方法似乎也可以做。QAQ

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
//int a[maxx];
struct pp{
    int p,d;
}a[maxx];
int b[maxx];
int ans = 0,cnt = 0,pos = 0;
int l = 0,r = 0;
vector<pp> ve;

int cmp(pp a,pp b)
{
    if(a.d != b.d) return a.d < b.d;
    return a.p > b.p;
}
int cmp1(pp a,pp b){
    if(a.p != b.p) return a.p<b.p;
    return a.d < b.d;
}

int dp[maxx][20];
int mm[maxx];

void initrmq(int n)
{
    mm[0] = -1;
    for(int i = 1; i <= n; i++)
    {
        mm[i] = ((i&(i-1)) == 0)?mm[i-1]+1:mm[i-1];
        dp[i][0] = a[i].p;
    }
    for(int j = 1; j <= mm[n]; j++){
        for(int i = 1; i + (1<<j) -1 <= n; i++){
            dp[i][j] = min(dp[i][j-1],dp[i+(1<<(j-1))][j-1]);
        }
    }
}
int rmq(int x,int y)
{
    int k = mm[y-x+1];
    return min(dp[x][k],dp[y-(1<<k)+1][k]);
}


int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/5-数据结构/rmq区间查询/st/hdu3193.txt","r",stdin);
#endif
    while(~scanf("%d",&n))
    {
        ve.clear();
        for(int i = 1; i <= n; i++)
        {
            scanf("%d%d",&a[i].p,&a[i].d);
        }
        sort(a+1,a+n+1,cmp);
        initrmq(n);
//        ve.push_back(a[1]);
        for(int i = 1; i <= n; i++)
        {
            int t = rmq(1,i);
//            printf("t = %d   %d %d\n",t,a[i].p,a[i].d);
            if(t >= a[i].p) ve.push_back(a[i]);
        }
        printf("%d\n",ve.size());
        sort(ve.begin(),ve.end(),cmp1);
        for(int i = 0; i < ve.size(); i++){
            printf("%d %d\n",ve[i].p,ve[i].d);
        }
    }
    return 0;
}

```



### 二维

尽是些板子题，不贴也罢？

有点感觉就行了。

## 树状数组

树状数组有着代码短小，常数小（快）的特点，所以能用树状数组的地方尽量就用。

BIT模板

```c++
int lowbit(int x)
{
    return x & (-x);
}
void add(int x,int add)//一维
{
    while(x <= n)
    {
        a[x]+=add;
        x+=lowbit(x);
    }
}
int sum(int x)
{
    int s=0;
    while(x > 0)
    {
        s += a[x];
        x -= lowbit(x);
    }
    return s;
}
```

```c++
void modify(int x,int y,int data)//二维
{
    for(int i=x;i<MAXN;i+=lowbit(i))
        for(int j=y;j<MAXN;j+=lowbit(j))
            a[i][j]+=data;
}
int get_sum(int x,int y)
{
    int res=0;
    for(int i=x;i>0;i-=lowbit(i))
        for(int j=y;j>0;j-=lowbit(j))
            res+=a[i][j];
    return res;
}
```

```c++
void add(int x,int y){
    for(int i=x;i<=n;i+=i&(-i)) 
    {
      	c1[i]+=y;
	  	c2[i]+=(long long)x*y;
    }//给差分数组中的位置x加上y
long long sum(int x){//查询前x项的和
    long long ans(0);
    for(int i=x;i;i-=i&(-i)) ans+=(x+1)*c1[i]-c2[i];
    return ans;
}
```





### 逆序对

> 逆序对仅在数组中有意义，对于一个包含n个非负整数的数组a，如果有i<j，且a[i]>a[j]，则称（a[i]，A[j]）为数组a中的一个逆序对

关于逆序对.....离散化

```c++
struct Node{
    int v;
    int order;
}a[maxx];

int cmp(Node a, Node b){
    return a.v < b.v;
}

int lowbit(int x)
{
    return x & (-x);
}
void add(int x,int add)//一维
{
    while(x <= n)
    {
        c[x]+=add;
        x+=lowbit(x);
    }
}
int sum(int x)
{
    int s=0;
    while(x > 0)
    {
        s += c[x];
        x -= lowbit(x);
    }
    return s;
}

int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/5-数据结构/bit树状数组/hdu1394.txt","r",stdin);
#endif
    scanf("%d",&n);
    for(int i = 1; i <= n; i++)
    {
        scanf("%d",&a[i].v);
        a[i].order = i;
    }
    sort(a+1,a+n+1,cmp);
    memset(c,0,sizeof(c));
    for(int i = 1; i <= n; i++)
    {
        b[a[i].order] = i;
    }
    int ans = 0;
    for(int i = 1; i <= n; i++)
    {
        add(b[i],1);
        ans += i - sum(b[i]);
        printf("%d\n",ans);
    }
```

我还是有点绕不过来。在这里理一下

首先，离散化a，a按数值大小排序，然后依照在原a中的序号给b[序号] = i，所以b现在就是离散化过的原数组了（大概

然后用离散过的b[i]，每次在b[i]上更新1，然后每次求1~b[i]的和就能求出小于b[i]的值的数量，嗯~巧妙。这时i-sum(b[i])就是当前数字与前面所有数字形成的逆序对的个数

那么问题在于为什么要在线更新？离线就是错的吗？试了一下直接会出很大的数。

我想了一下，因为离线的话就会无视顺序了。就 1 3，会把1更到3的前面导致1 3成为一个逆序对。

#### hdu1394

求一个数列不停左移（溢出填进最右边）过程中，逆序对最少的数量

其实就是将第一个数移到最后一个数。这个操作会减去第一个数的所有逆序对(sum b[i]-1，这里不需要考虑顺序因为是第一个元素），然后加上所有大于第一个数的逆序对(n-sum(b[i]))就是更新过的数量了。

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
int c[maxx];
int b[maxx];
int ans = 0,cnt = 0,pos = 0;
int l = 0,r = 0;

struct Node{
    int v;
    int order;
}a[maxx];

int cmp(Node a, Node b){
    return a.v < b.v;
}

int lowbit(int x)
{
    return x & (-x);
}
void add(int x,int add)//一维
{
    while(x <= n)
    {
        c[x]+=add;
        x+=lowbit(x);
    }
}
int sum(int x)
{
    int s=0;
    while(x > 0)
    {
        s += c[x];
        x -= lowbit(x);
    }
    return s;
}

int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/5-数据结构/bit树状数组/hdu1394.txt","r",stdin);
#endif
    while(~scanf("%d",&n))
    {
        for(int i = 1; i <= n; i++)
        {
            scanf("%d",&a[i].v);
            a[i].order = i;
        }
        sort(a+1,a+n+1,cmp);
        memset(b,0,sizeof(b));
        memset(c,0,sizeof(c));
        for(int i = 1; i <= n; i++)
        {
            b[a[i].order] = i;
        }
        int ans = 0;
        for(int i = 1; i <= n; i++)
        {
            add(b[i],1);
            ans += i-sum(b[i]);
        }
        int A = ans;
        for(int i = 1; i <= n; i++)
        {
            ans += (n-sum(b[i])-sum(b[i])+1);
            A = min(ans,A);
        }
        printf("%d\n",A);
    }
    return 0;
}
```

#### hdu1556

每次给出l,r，使得区间+1。问每个单点的值。思路就a[l]++,a[r+1]--，统计一波前缀和或者直接用树状数组维护

# 主席树

说实话这种数据结构我不是很懂。话说的很清楚：每个操作只修改logn个点（修改的点和改变过的点），对于每个修改都记录下来，使用n个线段树的思想却只是使用了历史版本和一个logn线。

> 注意，这个线段树对一条线段，保存的是这个数字区间的出现次数，所以是可以互相加减的！还有，由于每棵线段树都要保存同样的数字，所以它们的大小、形态也都是一样的！这实在是两个非常好的性质，是平衡树所不具备的。
>
> 对于询问 (i,j)，我只要拿出 Tj 和 Ti-1，对每个节点相减就可以了。说的通俗一点，询问 i..j 区间中，一个数字区间的出现次数时，就是这些数字在 Tj 中出现的次数减去在 Ti-1 中出现的次数。
>
> 那么有修改操作怎么办呢？
>
> 如果将询问看成求一段序列的数字和，那么上面那个相当于求出了前缀和。加入修改操作后，就要用树状数组等来维护前缀和了。于是那个 “很好的性质” 又一次发挥了作用，由于主席树可以互相加减，所以可以用树状数组来套上它。做法和维护前缀和长得基本一样，不说了。

大体就是这样，也说不出什么东西，然而理解不到。做题。

#### poj2104

又拿出第k大来当模板....看完主席树就知道其实线段树就能搞这个题，离散后记录数本身，然后查询数的位置应该就行。

主席树的思路有些许的不同了，就成了建[1,1];[1,2]…[1,n]的所有的线段树，然后直接把[1,l-1],[1,r]相减（某性质），后面一样。直接对现在这颗树二分查找就是第k大。

代码参照的uestc。

按照代码来理解的话，主席树里存的是离散化过的数据（就是有一个1就a[1]+1那种感觉），sum[i]表示[1,i]里有多少个数，

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
int ans = 0,cnt = 0,pos = 0;
int l = 0,r = 0;

vector<int> v;
const int N = 4e5+100;
int rt[N],ls[N*100],rs[N*100],sum[N*100];
int a[N],b[N],tot;
int getid(int x){
    return lower_bound(v.begin(),v.end(),x) - v.begin() + 1;
}
void build(int &o,int l,int r)
{
    o= ++tot,sum[o]=0;
    if(l==r) return ;
    int mid=(l+r)/2;
    build(ls[o],l,mid);
    build(rs[o],mid+1,r);
    return ;
}
void update(int &o,int l,int r,int last,int p,int v)
{//p是更新的点，last是上一个点，o是树根(?)
    o= ++tot;
    ls[o]=ls[last],rs[o]=rs[last];
    sum[o]=sum[last]+v;
    if(l==r) return ;
    int mid=(l+r)/2;
    if(p<=mid) update(ls[o],l,mid,ls[last],p,v);
    else update(rs[o],mid+1,r,rs[last],p,v);
    return ;
}
int vis[1000000+10];
int query(int x,int y,int l,int r,int k)
{
    if(l==r)
    {
        return l;
    }
    int mid=(l+r)/2;
    int s = sum[ls[y]]-sum[ls[x]];
    if(k<=s) return query(ls[x],ls[y],l,mid,k);
    else return query(rs[x],rs[y],mid+1,r,k-s);
}

int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/5-数据结构/zxs主席树/poj2104.txt","r",stdin);
#endif
    scanf("%d%d",&n,&m);
    for(int i = 1; i <= n; i++)
    {
        scanf("%d",&a[i]);
        v.push_back(a[i]);
    }
    sort(v.begin(),v.end());
    v.erase(unique(v.begin(),v.end()),v.end());
    build(rt[0],1,n);
    for(int i = 1; i <= n; i++)
    {
        update(rt[i],1,n,rt[i-1],getid(a[i]),1);
    }
    for(int i = 0; i < m; i++)
    {
        int l,r,k;
        scanf("%d%d%d",&l,&r,&k);
        int ans = v[query(rt[l-1],rt[r],1,n,k)-1];
        printf("%d\n",ans);
    }
    return 0;
}
```



#### spoj-dquery

题意：对于[i,j]统计其中不同数字的个数。

主席树里就是[1,n]不同数字的个数。所以对于每个查询[l,r]，对于第r颗树的从l开始的元素数。

更新主席树的时候，如果之前出现过这个数就在前面那颗树的位置-1，在现在的位置+1。

写的时候还是参照了别人的代码，query那一块不是很好理解。

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
int ans = 0,cnt = 0,pos = 0;
int l = 0,r = 0;

vector<int> v;
const int N = 4e5+100;
int rt[N],ls[N*100],rs[N*100],sum[N*100];
int a[N],b[N],tot,q;
int getid(int x){
    return lower_bound(v.begin(),v.end(),x) - v.begin() + 1;
}
void build(int &o,int l,int r)
{
    o= ++tot,sum[o]=0;
    if(l==r) return ;
    int mid=(l+r)/2;
    build(ls[o],l,mid);
    build(rs[o],mid+1,r);
    return ;
}
void update(int &o,int l,int r,int last,int p,int v)
{//p是更新的点，last是上一个点，o是树根(?)
    o= ++tot;
    ls[o]=ls[last],rs[o]=rs[last];
    sum[o]=sum[last]+v;
    if(l==r) return ;
    int mid=(l+r)/2;
    if(p<=mid) update(ls[o],l,mid,ls[last],p,v);
    else update(rs[o],mid+1,r,rs[last],p,v);
    return ;
}
int vis[1000000+10];
int query(int c,int pos,int l,int r)
{
    if(l==r)
    {
        return sum[c];
    }
    int mid=(l+r)/2;
//    int s = sum[ls[y]]-sum[ls[x]];
    if(pos<=mid)  return sum[rs[c]] + query(ls[c],pos,l,mid);
    else  return query(rs[c],pos,mid+1,r);
}


int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/5-数据结构/zxs主席树/spoj-dquery.txt","r",stdin);
#endif
    while(~scanf("%d",&n))
    {
        for(int i = 1; i <= n; i++)
        {
            scanf("%d",&a[i]);
            v.push_back(a[i]);
        }
        sort(v.begin(),v.end());
        v.erase(unique(v.begin(),v.end()),v.end());
        build(rt[0],1,n);
        for(int i = 1; i <= n; i++)
        {
            int tmp = 0;
            if(!vis[getid(a[i])])
            update(rt[i],1,n,rt[i-1],i,1);
            else
            {
                update(tmp,1,n,rt[i-1],vis[getid(a[i])],-1);
                update(rt[i],1,n,tmp,i,1);
            }
            vis[getid(a[i])] = i;
        }
        scanf("%d",&q);
        while(q--)
        {
            scanf("%d%d",&l,&r);
            ans = query(rt[r],l,1,n);
            printf("%d\n",ans);
        }
    }
    return 0;
}
```

