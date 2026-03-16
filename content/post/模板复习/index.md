---
title: "模板复习"
description: "模板复习"
date: 2026-03-03T17:06:21+08:00
image: cover.png
math: true
categories:
    - 记录（OI 与数学）
---

## 图论

### 单源最短路

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

#### prim

```cpp
```

### 强连通分量

[[图论与代数结构 701] 强连通分量](https://www.luogu.com.cn/problem/B3609)

#### tarjan

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=1e4+5;
ll n,m,cnt,sc;
vector<ll> g[N],ans[N];
ll dfn[N],ins[N],low[N],scc[N],vis[N];
stack<ll> s;

void tarjan(ll u){
    dfn[u]=low[u]=++cnt;
    s.push(u);
    ins[u]=1;
    for (ll v:g[u]){
        if (!dfn[v]){
            tarjan(v);
            low[u]=min(low[u],low[v]);
        }
        else if (ins[v]) low[u]=min(low[u],dfn[v]);
    }
    if (dfn[u]==low[u]){
        sc++;
        while (s.top()!=u){
            ll v=s.top();
            s.pop();
            scc[v]=sc;
            ins[v]=0;
        }
        scc[u]=sc;
        ins[u]=0;
        s.pop();
    }
}

int main(){
    scanf("%lld%lld",&n,&m);
    for (int i=1;i<=m;i++){
        ll u,v;
        scanf("%lld%lld",&u,&v);
        g[u].push_back(v);
    }
    for (int i=1;i<=n;i++){
        if (!dfn[i]) tarjan(i);
    }
    printf("%lld\n",sc);
    for (int i=1;i<=n;i++) ans[scc[i]].push_back(i);
    for (int i=1;i<=n;i++){
        if (vis[scc[i]]) continue;
        vis[scc[i]]=1;
        for (ll res:ans[scc[i]]) printf("%lld ",res);
        putchar('\n');
    }
    return 0;
}
```

### 网络流

[【模板】网络最大流](https://www.luogu.com.cn/problem/P3376)

#### 网络最大流（EK）

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=2e2+5;
const int M=5e3+5;
ll n,m,s,t,ans;
ll tot=1,h[N],dis[N],pre[N],vis[N];
struct node{
    ll v,w;
    ll nxt;
}e[M<<1];

void add(ll u,ll v,ll w){
    e[++tot]=(node){v,w,h[u]};
    h[u]=tot;
}

bool bfs(){
    memset(vis,0,sizeof vis);
    queue<ll> q;
    q.push(s);
    vis[s]=1;
    dis[s]=1e14;
    while (!q.empty()){
        ll u=q.front();
        q.pop();
        for (int i=h[u];i;i=e[i].nxt){
            ll v=e[i].v,w=e[i].w;
            if (vis[v]) continue;
            if (!w) continue;
            vis[v]=1;
            dis[v]=min(dis[u],w);
            pre[v]=i;
            q.push(v);
            if (v==t) return true;
        }
    }
    return false;
}

void update(){
    ll u=t;
    while (u!=s){
        ll v=pre[u];
        e[v].w-=dis[t];
        e[v^1].w+=dis[t];
        u=e[v^1].v;
    }
    ans+=dis[t];
}

int main(){
    scanf("%lld%lld%lld%lld",&n,&m,&s,&t);
    for (int i=1;i<=m;i++){
        ll u,v,w;
        scanf("%lld%lld%lld",&u,&v,&w);
        add(u,v,w);
        add(v,u,0);
    }
    while (bfs()) update();
    printf("%lld\n",ans);
    return 0;
}
```

#### 网络最大流（Dinic+当前弧优化）

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int N=2e2+5;
const int M=5e3+5;
const ll INF=1e14;
ll n,m,s,t,ans;
ll tot=1,h[N],dis[N],cur[N];
struct node{
    ll v,w;
    ll nxt;
}e[M<<1];

void add(ll u,ll v,ll w){
    e[++tot]=(node){v,w,h[u]};
    h[u]=tot;
}

bool bfs(){
    for (int i=1;i<=n;i++) dis[i]=INF,cur[i]=h[i];
    queue<ll> q;
    q.push(s);
    dis[s]=0;
    while (!q.empty()){
        ll u=q.front();
        q.pop();
        for (int i=h[u];i;i=e[i].nxt){
            ll v=e[i].v,w=e[i].w;
            if (w&&dis[v]==INF){
                q.push(v);
                dis[v]=dis[u]+1;
            }
        }
    }
    return dis[t]!=INF;
}

ll dfs(ll u,ll sum){
    if (u==t) return sum;
    ll res=0;
    for (ll &i=cur[u];i&&sum;i=e[i].nxt){
        ll v=e[i].v,w=e[i].w;
        if (w&&dis[v]==dis[u]+1){
            ll k=dfs(v,min(sum,w));
            if (!k) dis[v]=INF;
            e[i].w-=k;
            e[i^1].w+=k;
            res+=k;
            sum-=k;
        }
    }
    return res;
}

int main(){
    scanf("%lld%lld%lld%lld",&n,&m,&s,&t);
    for (int i=1;i<=m;i++){
        ll u,v,w;
        scanf("%lld%lld%lld",&u,&v,&w);
        add(u,v,w);
        add(v,u,0);
    }
    while (bfs()) ans+=dfs(s,INF);
    printf("%lld\n",ans);
    return 0;
}
```

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