---
title: "洛谷 3 月月赛 I"
description: "洛谷 3 月月赛 I 记录"
date: 2026-03-01T17:51:34+08:00
image: cover.jpg
math: true
categories:
    - OI与数学
    - 记录
tags:
    - 题解
---

比赛链接：
- div1：[https://www.luogu.com.cn/contest/309087](https://www.luogu.com.cn/contest/309087)
- div2：[https://www.luogu.com.cn/contest/309086](https://www.luogu.com.cn/contest/309086)     

100+100+100+0+0+0

## A.[「Stoi2037」晴天](https://www.luogu.com.cn/problem/P15545)
没什么好说的
```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=1e6+5;
ll n,s,x,p,ans=-1;
ll a[N];

int main(){
    scanf("%lld%lld%lld",&n,&s,&x);
    for (int i=1;i<=n;i++){
        scanf("%lld",&a[i]);
        if (a[i]!=-1) p+=x-a[i];
        if (p>=s) {
            ans=i;
            break;
        }
    }
    printf("%lld\n",ans);
    return 0;
}
```

## B.[「Stoi2037」七里香](https://www.luogu.com.cn/problem/P15546)
数学题和简单贪心，下文我们统一把 $a_i^{'}$ 写作 $a_i$，观察原式子：
$$
((j-1)k+a_j)-((i-1)k+a_i)
$$
$$
=(j-i)k+(a_j-a_i)
$$

注意到对于每一对 $(i,j)$ 会出现 $i(n-j+1)$ 次，总的计算得到：
$$
Ans=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n} i(n-j+1)[(j-i)k+(a_j-a_i)]
$$
$$
=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n} i(n-j+1)(j-i)k+\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}i(n-j+1)(a_j-a_i)
$$
第一部分是一个定值，因此只需最大化第二部分。我们考虑计算求和完之后 $a_i$ 的系数 $c_i$，不难计算得到：
$$
c_i=\sum_{m=1}^{i-1}m(n-i+1)-\sum_{m=i+1}^{n}i(n-m+1)
$$ 
$$
=\frac{i(n-i+1)(2i-n-1)}{2}
$$
因此答案转化为：
$$
Ans=k \sum_{i=1}^{n} i \cdot c_i+\sum_{i=1}^{n}c_i \cdot a_i
$$
考虑怎么重排 $a_i$，只看第二部分，如果把 $c_i$ 从小到大排序，要想最大化此值，就让 $a_i$ 也从小到大排序去逐个相乘。最后的答案要使用 `__int128`。
```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=1e5+5;
ll n,k,a[N],c[N];

void print(__int128 x) {
    if (x<0) {
        putchar('-');
        x=-x;
    }
    if (x>9) print(x/10);
    putchar(x%10+'0');
}

int main(){
    scanf("%lld%lld",&n,&k);
    for (int i=1;i<=n;i++) scanf("%lld",&a[i]);
    sort(a+1,a+1+n);
    __int128 ans=0;
    for (int i=1;i<=n;i++){
        c[i]=i*(n-i+1)*(2*i-n-1)/2;
        ans+=(__int128)c[i]*i*k;
    } 
    sort(c+1,c+1+n);
    for (int i=1;i<=n;i++) ans+=(__int128)a[i]*c[i];
    print(ans);
    return 0;
}
```

## C.[「Stoi2037」白色风车](https://www.luogu.com.cn/problem/P15547)
考虑什么样的图可以使得点 $x$ 和点 $y$ 达到**永远**，什么样的图不能。

1. 若两点不在一个连通图上，显然不可能。
2. 若两点所在的连通块内没有环（一棵树），也是不可能的。    

现在的连通块内存在环，注意 Subtask 3 提到了二分图。如果这张图不是二分图，说明存在奇环，只需要让这两棋子走到环中，再对向走，一定有一个方向可以使得两棋子走到同一条边上，因为棋子 $x$ 可以回头，此时两棋子只需要再同向走就能达到**永远**。如果这个图是二分图，如果两棋子颜色一样（位于同一侧），两棋子无论怎么走颜色都一样，始终无法处于一条边上，因此无解；如果两棋子的颜色不一样，且二分图有环（没有环的情况已经被我们排除了），只需要让两棋子各自走到环中，然后对向走可以使得两棋子走到一条边上，棋子 $x$ 再往回走之后一起同向走便可实现**永远**。

3. 若两点所在的连通块不是二分图且有环，说明可以。
4. 若两点所在的连通块是二分图且有环，如果两点颜色一样，则无解。否则就可以。

这四个判断步骤走一遍就能涵盖所有情况。用 DFS 判断是否是二分图，同时判断图是否连通，再用连通块内点的个数和点的度数之和判断是不是树。复杂度是 $O(n+m)$。

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=1e6+5;
ll Tid,T;
ll n,m,x,y;
vector<ll> g[N];
ll vis[N],notb,color[N],du[N],cnt;

void dfs(ll u){
    cnt++;
    vis[u]=1;
    for (ll v:g[u]){
        if (vis[v]) {
            if (color[v]==color[u]) notb=1;
        }
        else {
            color[v]=color[u]^1;
            dfs(v);
        }
        du[u]++;
    }
}

int main(){
    scanf("%lld%lld",&Tid,&T);
    while (T--){
        scanf("%lld%lld%lld%lld",&n,&m,&x,&y);
        for (int i=0;i<=n;i++) g[i].clear(),vis[i]=0,color[i]=0,du[i]=0;
        notb=0,cnt=0;
        for (int i=1;i<=m;i++){
            ll u,v;
            scanf("%lld%lld",&u,&v);
            g[u].push_back(v);
            g[v].push_back(u);
        }
        dfs(x);
        for (int i=1;i<=n;i++) du[0]+=du[i];
        if (!vis[y]) puts("No");
        else if (2*(cnt-1)==du[0]) puts("No");
        else if (notb) puts("Yes");
        else {
            if (color[x]==color[y]) puts("No");
            else puts("Yes");
        }
    }
    return 0;
}
```