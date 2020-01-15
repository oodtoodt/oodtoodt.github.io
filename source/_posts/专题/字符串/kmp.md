title: kmp

tags:
- special
- kmp

categories:
- acm

date: 2017-07-19 17:37:00

---

>kmp....
>[直接看这个！](http://www.cnblogs.com/zhangtianq/p/5839909.html)
>以上

<!--more-->

---
###kmp
直接从模版题开始吧。
##### hdu1711
给出a串主串b串匹配串，求a串中第一次与b串完全重合的位置
相当于模版 但是用char串会有蜜汁re 换成int就直接过了，黑人问号.jpg。存疑好了
代码不贴了，在第一次匹配到的时候return就行
##### hdu1686
求匹配次数
套模版代码不贴
##### hdu2087
求匹配次数。小布条上裁剪。所以直接在每次动匹配串位置的时候提前位置
从j = next[j] 变为 j = next[j] - m + 1.（其实就是清0，不更新到前面那个j而是从新开始。意会去吧
##### Hdu3746
一串珍珠，问将其首尾相连变为某种串的循环，需要向尾部添加多少珠子？
next数组的应用。应使用未优化过的next数组求法，可以求出最长前缀后缀
举个例子，abcdabd 对应-1000012，可以看作0000120右移。next[len]就是右移的那个0。如此便可求出....最长重复。
len - next[len] = 循环串长度。
若循环长度与len可以相整除，则啥都不用加。
否则用循环串长度 - next[len] % 循环串长度 就是要加的数量。abcgabcga 4 - 5 % 4 注意%不可省。n
代码如上所述。
##### hdu1358
给出一个字符串，问从头到尾的所有位置里，字符串前缀出现重复的次数和位置。
上一道题的加强版。（一开始并不能看出来
x-next[x] = 循环串长度。只要此长度与x相整除，则循环（和上就是一样。
注意用next[] == 0 / -1 continue优化一下，不然可能有点慢(不过错误居然是ole....
```c++
int kmpnext[maxx];
void getnext(const char *s,int m)
{
    int i,j;
    j = kmpnext[0] = -1;
    i = 0;
    while(i != m)
    {
        if(j == -1 || s[i] == s[j])
        kmpnext[++i] = ++j;
        else
        j = kmpnext[j];
    }
}


int main()
{
#ifdef LOCAL
    freopen("/Users/ecooodt/Desktop/c++ and acm/special--专题/2-字符串/kmp/hdu1358.txt","r",stdin);
#endif
    while(scanf("%d",&n) && n != 0)
    {
        getchar();
        for(int i = 0; i < n; i++)
        {
            scanf("%c",&a[i]);
        }
        int len = strlen(a);
        getnext(a,len);
        printf("Test case #%d\n",++cnt);
        for(int i = 2; i <= len; i++)
        {
            if(kmpnext[i] == -1 || kmpnext[i] == 0) continue;
            int j = i - kmpnext[i];
            if(i % j == 0)
            {
                printf("%d %d\n",i,i/j);
            }
        }
        printf("\n");
    }
    return 0;
}
```



小结：kmp的最初级的应用就是作为两个串的比较。但是更多的时候还是关于next数组的一个运用。
next存的可以看成是与前缀相同的长度

扩展kmp今天看了看，没明白，回头做几个题算了。

继续贴题。
##### hust 1010
题意略 最后也是求重复段，用len - next[len].
##### poj 2406
因为是考虑整个串所以直接求循环次数 len / (len - next[len]) 注意不%要输出1
##### poj 2752
这个题求一个串前缀和后缀相同的所有长度。
起初我以为是扩展kmp，后来发现是kmp
首先next[len]一定是前缀相同的最大长度。
然后视这个最大长度串为新串，递归就好了。
至于不在比这个长度更大的串里找，是因为如果存在一个略比这个最大小的next一定在这个next之中成后缀。emm，自己画个图感受一下就好了

这里就涉及了一个很有用的一段 ：
```c++
for(int i = len; i != 0;)
{
    sum[cnt++] = kmpnext[i];
    i = kmpnext[i];
}
```

