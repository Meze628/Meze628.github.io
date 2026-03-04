---
title: "模板复习"
description: "模板复习"
date: 2026-03-03T17:06:21+08:00
image: cover.jpg
math: true
categories:
    - OI与数学
    - 记录
---

## 图论

### 最短路

#### dijkstra

[P4779 【模板】单源最短路径（标准版）](https://www.luogu.com.cn/problem/P4779)

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=1e5+5;
ll n,m,s;
ll dis[N],vis[N];
struct edge{
    ll v,w;
};
vector<edge> g[N];
struct node{
    ll dis,u;
    bool operator>(const node &b) const{
        return b.dis<dis;
    }
};
priority_queue<node,vector<node>,greater<node> > q;

void dijkstra(){
    memset(dis,0x3f,sizeof dis);
    memset(vis,0,sizeof vis);
    dis[s]=0;
    q.push((node){0,s});
    while (!q.empty()){
        ll u=q.top().u;
        q.pop();
        if (vis[u]) continue;
        vis[u]=1;
        for (edge ed:g[u]){
            ll v=ed.v,w=ed.w;
            if (dis[v]>dis[u]+w){
                dis[v]=dis[u]+w;
                q.push((node){dis[v],v});
            }
        }
    }
}

int main(){
    scanf("%lld%lld%lld",&n,&m,&s);
    for (int i=1;i<=m;i++){
        ll u,v,w;
        scanf("%lld%lld%lld",&u,&v,&w);
        g[u].push_back((edge){v,w});
    }
    dijkstra();
    for (int i=1;i<=n;i++) printf("%lld ",dis[i]);
    return 0;
}
```


#### spfa

[P3371 【模板】单源最短路径（弱化版）](https://www.luogu.com.cn/problem/P3371)

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=1e4+5;
ll n,m,s;
ll inq[N],dis[N];
struct edge{
    ll v,w;
};
vector<edge> g[N];
queue<ll> q;

void spfa(){
    for (int i=1;i<=n;i++) dis[i]=INT_MAX;
    memset(inq,0,sizeof inq);
    q.push(s);
    dis[s]=0;
    inq[s]=1;
    while (!q.empty()){
        ll u=q.front();
        q.pop();
        inq[u]=0;
        for (edge ed:g[u]){
            ll v=ed.v,w=ed.w;
            if (dis[v]>dis[u]+w){
                dis[v]=dis[u]+w;
                if (!inq[v]) inq[v]=1,q.push(v);
            }
        }
    }
}

int main(){
    scanf("%lld%lld%lld",&n,&m,&s);
    for (int i=1;i<=m;i++){
        ll u,v,w;
        scanf("%lld%lld%lld",&u,&v,&w);
        g[u].push_back((edge){v,w});
    }
    spfa();
    for (int i=1;i<=n;i++) printf("%lld ",dis[i]);
    return 0;
}
```

### 最小生成树
[P3366 【模板】最小生成树](https://www.luogu.com.cn/problem/P3366)

#### 

#### Kruskal

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int M=2e5+5;
ll n,m,p[M],ans,cnt;
struct Edge{
    ll u,v,w;
    bool operator<(const Edge &b) const {
        return w<b.w;
    }
}g[M];

ll find(ll x){
    if (p[x]!=x) p[x]=find(p[x]);
    return p[x];
}

int main(){
    scanf("%lld%lld",&n,&m);
    for (int i=1;i<=m;i++){
        ll u,v,w;
        scanf("%lld%lld%lld",&u,&v,&w);
        g[i]=(Edge){u,v,w};
    }
    for (int i=1;i<=n;i++) p[i]=i;
    sort(g+1,g+1+m);
    for (int i=1;i<=m;i++){
        ll u=g[i].u,v=g[i].v,w=g[i].w;
        ll x=find(u),y=find(v);
        if (x!=y) p[x]=y,ans+=w,cnt++;
    }
    if (cnt==n-1) printf("%lld\n",ans);
    else printf("orz");
    return 0;
}
```
### 强连通分量

#### tarjan

```cpp

```

### 网络流

## 数据结构

### 并查集

[P3367 【模板】并查集](https://www.luogu.com.cn/problem/P3367)

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=2e5+5;
ll n,m,p[N];

ll find(ll x){
    if (p[x]!=x) p[x]=find(p[x]);
    return p[x];
}

int main(){
    scanf("%lld%lld",&n,&m);
    for (int i=1;i<=n;i++) p[i]=i;
    for (int i=1;i<=m;i++) {
        ll z,x,y;
        scanf("%lld%lld%lld",&z,&x,&y);
        if (z==1) p[find(x)]=find(y);
        if (z==2){
            if (p[find(x)]==p[find(y)]) puts("Y");
            else puts("N");
        }
    }
    return 0;
}
```

## 数学

## 字符串

## 其他