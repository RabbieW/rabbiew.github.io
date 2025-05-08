---
title: AtCoder Beginner Contest A-Ex 题解
date: 2022-08-05 11:58:46
tags:
    - 题解
    - AtCoder
mathjax: true
categories: OI
description: AtCoder Beginner Contest 261 A-Ex 题解
---

# A. Intersection

两个区间相交的条件：$L_2 > R_1$ 或 $L_1 > R_2$。

所以答案为 $\max\{L_2 - R_1,L_1 - R_2\}$。

# B. Tournament Result

答案合法当且仅当 $A_{i,j} = A_{j,i} | j,i \in [1,N] \operatorname{and} j \neq i$。

直接暴力判断即可，时间复杂度 $O(n^2)$。

# C. NewFolder(1)

用 `map` 记录每个文件名出现的次数，直接 $O(n \log n)$ 求解。

# D. Flipping and Bonus

考虑 DP。

用 $f(i,j)$ 表示当前枚举到第 $i$ 次掷硬币，计数器显示 $j$ 时最多赚多少钱。

令 $D(C_i) = Y_i$，

显然转移方程为
$$
f(0,0) = 0
$$

$$
f(i,j) =
\left \{
\begin{aligned}
& f(i-1,j - 1) + X_i + D(j) & &j > 0 \\
& \max \limits_{k=0} ^{i-1} f(i-1,k) & &j=0
\end{aligned} 
\right.
$$

最后答案为 $\max \limits _{i=0} ^n \{f(n,i)\}$。

时间复杂度为 $O(n^2)$，记得开 `long long`。

# E. Many Operations

对于每一个二进制位可以分别考虑。

对于每一**轮**（很多次）操作，可以 $O(1)$ 计算出这一轮操作前为 $0/1$ ，操作后变为什么：

用 $f(i,0/1)$ 表示第 $i$ 轮操作前为 $0/1$，操作后变为什么；$g(i,0/1)$ 表示 $0/1$ 经过第 $i$ **种**操作后变为什么。

则 $f(i,0) = g(i,f(i - 1,0))$，$f(i,1) = g(i,f(i - 1,1))$。

显然 $f$ 的第一位是可以滚掉的（不滚也行？）。

只需要记录上一轮操作后**最开始**（$C$）的第 $k$ 位变为什么，即可 $O(1)$ 求出这一轮操作后**最开始**的第 $k$ 位变为什么，并更新答案。

时间复杂度 $O(n \log C)$。

代码（有点丑）：

```cpp
#include<iostream>
#include<cstdio>
using namespace std;

int n,c;
int t[200010],a[200010],f[3],ans[200010],g[3];

int main()
{
    scanf("%d%d",&n,&c);
    for (int i = 1;i <= n;i ++) scanf("%d%d",t + i,a + i);
    g[1] = 1;// g 表示初始值为 0/1 经过某一轮操作后变为什么
    for (int i = 0,nw = (c >> i) % 2;i <= 30;++ i,nw = (c >> i) % 2,g[0] = 0,g[1] = 1)
        for (int j = 1;j <= n;j ++)
        {
            // nw 记录第 i 位现在是什么
            int tmp = (a[j] >> i) % 2; // 求 a[j] 的第 i 位
            if (t[j] == 1) f[0] = 0,f[1] = (1 & tmp);
            else if (t[j] == 2) f[0] = (0 | tmp),f[1] = 1;
            else f[0] = (0 ^ tmp),f[1] = (1 ^ tmp);
            // f[0/1] 即题解中的 g(i,0/1)
            g[0] = f[g[0]],g[1] = f[g[1]];
            nw = g[nw];
            ans[j] |= (nw << i); // 更新答案
        }
    for (int i = 1;i <= n;i ++) printf("%d\n",ans[i]);
}
```

# F. Sorting Color Balls

考虑所有数的颜色都不一样，答案即为逆序对个数。

若有数的颜色一样，那么答案就应减去这种颜色的数组成的逆序对个数。

所以只需要统计总逆序对个数和每种颜色的逆序对个数，时间复杂度为 $O(n \log n)$。

记得开 `long long`。

可以先把数用 `vector` 按颜色分类，再求解。

代码（用了树状数组）：

```cpp
#include<iostream>
#include<cstdio>
#include<vector>
#define int long long
using namespace std;

int n;
int val[300010],c[300010];
int tree[300010];
vector <int> v[300010];
int ans;

int lowbit(int x)
{
    return x & (-x);
}

void upd(int x,int y)
{
    while (x <= n)
    {
        tree[x] += y;
        x += lowbit(x);
    }
}

int query(int x)
{
    int res = 0;
    while (x)
    {
        res += tree[x];
        x -= lowbit(x);
    }
    return res;
}

signed main()
{
    scanf("%lld",&n);
    for (int i = 1;i <= n;i ++) scanf("%lld",c + i),v[c[i]].push_back(i);// 将数按颜色分类
    for (int i = 1;i <= n;i ++) scanf("%lld",val + i);
    for (int i = 1;i <= n;i ++)
        ans += query(n) - query(val[i]),upd(val[i],1);
    for (int i = 1;i <= n;i ++) upd(val[i],-1);
    for (int i = 1;i <= n;i ++)
    {
        for (int j = 0;j < v[i].size();j ++)
        {
            int x = v[i][j];
            ans -= query(n) - query(val[x]);
            upd(val[x],1);
        }
        for (int j = 0;j < v[i].size();j ++)
        {
            int x = v[i][j];
            upd(val[x],-1);
        }
    }
    printf("%lld\n",ans);
}
```

# G. Replace

反向思考，题目转化为能否从 $T$ 到 $S$，操作全部反过来（字符串 $\to$ 字母）。

令 $S(l,r)$ 表示 $S_lS_{l+1}\dots S_r$，$len(S)$ 表示 $|S|$，$n = len(T)$，$m = ken(S)$。

考虑区间 DP，$f(i,j,k)$ 表示 $T(i,j)$ 变为字母 $k$ 的最小操作数，$g(i,j,k,l)$ 表示 $T(i,j) \to A_k(1,l)$ 的最小操作数。

我们先用替换后长度减小的操作更新，再用替换后长度不变（$|A_i| = 1$）的操作更新，来保证多种情况叠加的正确转移。

对于长度为 $1$ 的区间，有 $f(i,j,T_i) = 0$。

在求 $g(i,j,k,l)$ 时，把 $[i,j]$ 划分成 $[i,p]$ 和 $[p + 1,j]$ 两个区间，则 $S(i,p)$ 应变为 $A_k(1,l - 1)$，$S(p + 1,j)$ 应变为 $A_k(l,l)$。转移方程为：$g(i,j,k,l) = \min \limits _{p=i} ^{j-1} \{ g(i,p,k,l - 1) + f(p + 1,j,A_{k,l}) \}$。

而 $g(i,j,k,len(S))$ 表示把 $T(i,j)$ 变为 $A_k$ 的最小操作数。于是可以更新 $C_k$ 的答案：$f(i,j,C_k) = \min\{ f(i,j,C_k),g(i,j,k,len(k)) + 1 \}$。

对于区间 $[i,j]$，求出所有 $g(i,j,k,l)$ 后，可以求出 $f(i,j,k)$。

利用类似 Dijkstra 或 Floyd 的思想，把所有 $len(A_i) = 1$ 的 $A_i$ 和 $C_i$ 连一条权值为 $1$ 的有向边，把所有已知的 $f(i,j,k)$ 放进堆里，再类似求最短路把所有 $f(i,j,k)$ 求出来。

我们发现，在求 $g(i,j,k,l)$ 时，要用到 $g(i,j_0,k,l)$（$i \leq j_0 \leq j - 1$）和 $f(i_0,j_0,k)$ （$i < i_0 \leq j \leq n$）的答案，所以 $i$ 应倒序枚举。

接下来就只剩求答案了。用 $h(i,j)$ 表示 $T(1,i)$ 变为 $S(1,j)$ 的最小操作数。类似 $g$ 的转移方程，有 $h(i,j) = \min \limits _{p=0} ^{i-1} \{ h(p,j - 1) + f(p + 1,i,S_j) \}$。

答案即为 $h(n,m)$。

关于时间复杂度，对于区间 $[i,j]$，计算 $g(i,j,k,l)$ 的时间复杂度为 $O(Kn\max(|A_i|))$，计算 $f(i,j,k)$ 的时间复杂度为 $O((K + 26) \log 26)$；计算答案的复杂度为 $O(n ^2 m)$。

总的时间复杂度为 $O(n^3K\max(|A_i|) + n^2(K+26) \log 26 + n^2m)$，大约为 $50^5 = 312500000$，常数约为 $\frac{1}{12}$ ~ $\frac{1}{6}$（据官方题解所述），所以跑的过去。

答案最大为 $32000$，不用开 `long long`。

代码：

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<vector>
#include<queue>
#define Pair pair <int, int>
#define inf int(0x3f3f3f3f)
using namespace std;

char s[60],t[60],a[60][60],c[60];
int n,m,len[60],K;
int f[60][60][30],g[60][60][60][60],h[60][60];
vector <int> v[30];
priority_queue <Pair, vector <Pair>, greater <Pair> > q;

int main()
{
    scanf("%s",s + 1);
    scanf("%s",t + 1);
    n = strlen(t + 1),m = strlen(s + 1);
    scanf("%d",&K);getchar();
    for (int i = 1;i <= K;i ++)
    {
        scanf("%c%s",c + i,a[i] + 1);
        getchar();
        len[i] = strlen(a[i] + 1);
        if (len[i] == 1) v[a[i][1] - 'a'].push_back(c[i] - 'a');
    }
    memset(f,0x3f,sizeof(f));
    memset(g,0x3f,sizeof(g));
    for (int i = 1;i <= n;i ++)
        for (int j = i;j <= n;j ++)
            for (int k = 1;k <= K;k ++)
                g[i][j][k][0] = 0;
    for (int i = n;i >= 1;i --)
    {
        for (int j = i;j <= n;j ++)
        {
            if (i == j)
            {
                f[i][j][t[i] - 'a'] = 0;
                q.push(make_pair(0,t[i] - 'a'));
            }
            else
            {
                for (int k = 1;k <= K;k ++)
                {
                    if (len[k] == 1) continue;
                    for (int l = 1;l <= len[k];l ++)
                        for (int p = i;p < j;p ++)
                        {
                            if (g[i][p][k][l - 1] == inf || f[p + 1][j][a[k][l] - 'a'] == inf) continue;
                            g[i][j][k][l] = min(g[i][j][k][l],g[i][p][k][l - 1] + f[p + 1][j][a[k][l] - 'a']);
                        }
                    if (g[i][j][k][len[k]] + 1 < f[i][j][c[k] - 'a'])
                    {
                        f[i][j][c[k] - 'a'] = g[i][j][k][len[k]] + 1;
                        q.push(make_pair(f[i][j][c[k] - 'a'],c[k] - 'a'));
                    }
                }
            }
            while (q.size())
            {
                int nw = q.top().second,res = q.top().first;
                q.pop();
                if (res != f[i][j][nw]) continue;
                for (int p = 0;p < v[nw].size();p ++)
                    if (res + 1 < f[i][j][v[nw][p]])
                    {
                        f[i][j][v[nw][p]] = res + 1;
                        q.push(make_pair(res + 1,v[nw][p]));
                    }
            }
            for (int k = 1;k <= K;k ++) g[i][j][k][1] = f[i][j][a[k][1] - 'a'];
        }
    }
    memset(h,0x3f,sizeof(h));
    h[0][0] = 0;
    for (int i = 1;i <= n;i ++)
        for (int j = 1;j <= m;j ++)
            for (int k = 0;k < i;k ++)
                if (j) h[i][j] = min(h[i][j],h[k][j - 1] + f[k + 1][i][s[j] - 'a']);
    if (h[n][m] >= inf) printf("-1\n");
    else printf("%d\n",h[n][m]);
}
```

（终于结束了QAQ）

# Ex. Game on Graph

先考虑只判断游戏是否会无限进行下去。

用 $f(i,1/2)$ 表示现在在点 $i$，该 Arisa / Kasumi 走时游戏是否会无限进行，会则值为 $1$，否则为 $0$。

令 $out(i)$ 表示点 $i$ 的出度，

则有

$$
\left\{
\begin{aligned}
f(i,1) &= \max \limits _{j = son(i)} f(j,2) & &out(i) > 0\\
f(i,2) &= \max \limits _{j=son(i)} f(j,1) & &out(i) > 0 \\
f(i,1) &= f(i,2) = 0 & &out(i)=0
\end {aligned}
\right.
$$

时间复杂度 $O(M)$ 即可求出解。

考虑图为 DAG 的情况。

用 $g(i,1/2)$ 表示现在在点 $i$，该 Arisa / Kasumi 走时，在这之后经过的边的边权之和。

令 $dis(i,j)$ 表示点 $i,j$ 之间的距离，

则有

$$
\left\{
\begin{aligned}
g(i,1) &= \min \limits _{j=son(i)} \{ g(j,2) + dis(i,j) \} & &out(i) > 0 \\
g(i,2) &= \max \limits _{j=son(i)} \{ g(j,1) + dis(i,j) \} & &out(i) > 0 \\
g(i,1) &= g(i,2) = 0 & &out(i) = 0
\end{aligned}
\right.
$$

$g(i,1/2)$ 初始值为 $inf$，时间复杂度 $O(m)$ 即可求出解。

现在来考虑原题的情况。把上面的 DP 倒过来做，将所有边反向连接，从 $out(i) = 0$ 的点开始，类似 Dijkstra / 拓扑排序的思想更新答案，每次把更新完的点放进堆里，注意 $\min$ 和 $\max$ 的处理。

用 $g(i,1/2) = inf$ 表示游戏会无限进行，发现 $g$ 可以代替 $f$。

注意，$g(i,2)$ 的优先级应比 $g(i,1)$ 高（自己想一想？），所以在更新 $g(son(i),2)$ 的答案时，应等到所有可以到达 $son(i)$ 的点都更新完 $g(son(i),2)$ 的答案后，再把 $son(i)$ 放进堆里，不然会影响之后算出来的答案。要判断是否所有可以到达 $son(i)$ 的点都更新完 $g(son(i),2)$ 的答案，可以采用这样的策略：每次更新完 $g(son(i),2)$ 的答案后就把 $in(son(i)) \to in(son(i))-1$，当 $in(son(i))=0$ 时就把 $son(i)$ 入堆，原因显然（结合堆的性质，想一想每个点被放进堆的顺序）。这样做也可以去除环的影响。

最后的答案就是 $g(v,1)$。

这个 DP 的时间复杂度为 $O(M \log M)$。

代码（记得开 `long long`）：

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<queue>
#define inf (long long)(1e18)
#define ll long long
#define Pair pair <long long, pair <long long,long long> >
using namespace std;

ll n,m,v;
ll to[200010],val[200010],in[200010],nxt[200010],head[200010],cnt,p[200010];
ll f[200010][3];
priority_queue<Pair, vector <Pair>, greater <Pair> > q;

void add(int x,int y,int z)
{
    to[++ cnt] = y;
    val[cnt] = z;
    nxt[cnt] = head[x];
    in[y] ++;
    head[x] = cnt;
}

void dp()
{
    for (ll i = 1;i <= n;i ++,p[i] = 0)
        if (!in[i]) f[i][1] = f[i][2] = 0,q.push(make_pair(0,make_pair(i,1))),q.push(make_pair(0,make_pair(i,2)));
        else f[i][1] = f[i][2] = inf;
    while (q.size())
    {
        ll x = q.top().second.first,op = q.top().second.second,tmp = q.top().first;
        q.pop();
        if (op == 2) tmp = -tmp;
        if (f[x][op] != tmp) continue;
        if (op == 2)
        {
            for (ll i = head[x];i;i = nxt[i])
            {
                ll u = to[i],dis = val[i];
                if (f[u][1] > tmp + dis) f[u][1] = tmp + dis,q.push(make_pair(f[u][1],make_pair(u,1)));
            }
        }
        else
        {
            for (ll i = head[x];i;i = nxt[i])
            {
                ll u = to[i],dis = val[i];
                if (f[u][2] < tmp + dis || !p[u]) f[u][2] = tmp + dis;
                if (!p[u]) p[u] = 1;
                in[u] --;
                if (!in[u]) q.push(make_pair(-f[u][2],make_pair(u,2)));
            }
        }
    }
}

int main()
{
    scanf("%lld%lld%lld",&n,&m,&v);
    for (ll i = 1;i <= m;i ++)
    {
        ll x,y,z;
        scanf("%lld%lld%lld",&x,&y,&z);
        add(y,x,z);
    }
    dp();
    if (f[v][1] >= inf) printf("INFINITY\n");
    else printf("%lld\n",f[v][1]);
}
```
