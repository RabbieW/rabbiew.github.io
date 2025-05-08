---
title: 线段树 & 线段树合并优化 DP
tags: 
    - 笔记
    - DP
    - 数据结构
date: 2023-05-19 11:54:33
mathjax: true
description: 线段树和线段树合并优化 DP 的学习笔记
categories: OI
---

# 线段树优化 DP

有一些 DP 的初始化和转移操作可以转化为序列上 / 值域上的区间操作 / 区间查询问题，可以用线段树加速这些操作。

## 例题 1. [[NOIP1999 普及组] 导弹拦截 - 洛谷](https://www.luogu.com.cn/problem/P1020)

求序列的最长不上升 / 最长上升子序列，$1 \le n \le 10^5$，$1 \le a_i \le 5\times10^4$。

****

以最长不上升子序列为例。

朴素转移方程：$f_i = \max \limits_{j=0} ^{i - 1} (f_j+1)[a_j \ge a_i \lor j=0 ]$，$f_i$ 表示以 $a_i$ 结尾的最长子序列，从 $f_0$ 转移表示作为开头。这个 DP 时间复杂度为 $\mathcal{O} (n^2)$。

考虑优化。枚举到 $k$ 时，如果有 $i,j \le k$，$a_i = a_j$ 且 $f_i < f_j$，那么 $f_i$ 显然没用。由于值域很小，可以用 $g_i$ 记录 $\max \limits_{a_j=i} f_j$，转移方程变为 $f_i = \max \limits_{j=a_i}^{V} {g_j}$，在计算 $f_i$ 的同时更新 $g_{a_i}$ 即可，时间复杂度 $\mathcal{O}(nV)$。

发现转移时，相当于在 $[0,V]$ 中求了一次后缀最大值，可以用数据结构维护区间最大值，更新 $g_{a_i}$ 的操作转化为单点修改，可以用线段树或其他数据结构维护，时间复杂度 $\mathcal{O}(n\log V)$，可以通过。

## 例题 2. [Problem - 1334F - Codeforces](https://codeforces.com/problemset/problem/1334/F)

翻译：[Strange Function - 洛谷](https://www.luogu.com.cn/problem/CF1334F)

定义函数 $f$：$f(x)$ 为所有满足 $x_i>x_{1,2,\cdots,i-1}$ 的 $x_i$ 组成的序列，例如 $f[3,1,2,7,7,3,6,7,8]=[3,7,8]$。

给出两个序列 $a,b$，你可以删掉 $a$ 中的一些元素。删掉 $a_i$ 的代价为 $p_i$。你需要求出最小代价使得 $f(a)=b$ 或给出无解。

$1 \le |a| \le 5\times 10^5$，$b_{i-1} < b_i$。

----

设计状态 $f_{i,j}$ 表示现在考虑到第 $i$ 个序列 $a$ 中的元素，考虑完元素 $i$ 后新序列的目前最大值（有可能不在目前的新序列中）为 $b_j$，目前代价最小为 $f_{i,j}$。

设 $b_{k} \le a_i < b_{k+1}$。

先假设 $b_j$ 已经 / 将来一定会取到。

$p_i>0$ 时，若 $b_{j} \le a_i$ 即 $j \le k$，为了不改变目前最大值，$a_i$ 必须删除，$f_{i,j} \leftarrow f_{i-1,j}+p_i$；否则，$a_i$ 删不删都可以，$f_{i,j} \leftarrow f_{i-1,j}$。

$p_i < 0$ 时，$a_i$ 删除更优，$f_{i,j} \leftarrow f_{i-1,j}+p_i$。

为了保证最后 $b_j$ 可以取到，当 $a_i=b_j$ 时，$a_i$ 不能删除，$f_{i,j} = \min \limits_{v=0} ^j f_{i-1,v} = \min \limits_{v=0} ^{j} {f_{i,v} - p_i}$（减 $p_i$ 是因为 $v \le j$，在上面加上了 $p_i$，要把 $p_i$ 减回来）。

最终答案即为 $f_{|a|,|b|}$。

发现上面的几个式子可以转化为区间加、区间查询和单点修改，可以用线段树维护。时间复杂度 $\mathcal{O}(|a|\log |b|)$。

DP 的主要代码：

```cpp
signed main()
{
    scanf("%lld",&n);
    for (int i = 1;i <= n;i ++) scanf("%lld",a + i);
    for (int i = 1;i <= n;i ++) scanf("%lld",p + i);
    scanf("%lld",&m);
    for (int i = 1;i <= m;i ++) scanf("%lld",b + i),pl[b[i]] = i;
    build(1,0,m);
    for (int i = 1;i <= n;i ++)
    {
        int x = lower_bound(b + 1,b + m + 1,a[i]) - b;
        if (p[i] > 0) add(1,0,x - 1,p[i]);
        else add(1,0,m,p[i]);
        if (pl[a[i]]) upd(1,pl[a[i]],query(1,pl[a[i]] - 1) - p[i]);
    }
    // add 为区间加，query 为区间查询，upd 为单点修改
    if (query(1,m) < 1e15) printf("YES\n%lld\n",query(1,m));
    else printf("NO\n");
}
```

## 例题 3. [某位歌姬的故事 - 洛谷](https://www.luogu.com.cn/problem/P4229) / [Ex - Max Limited Sequence](https://atcoder.jp/contests/abc262/tasks/abc262_h)

~~其中前一道题实际上可以不用线段树优化 DP。~~

构造一个长度为 $n$ 的整数序列 $a$，要求 $1\le a_i \le A$，还有 $Q$ 条形如 $\max \{a_{l_i},a_{l_i+1},\cdots,a_{r_i}\}=m_i$ 的限制，问有多少种构造方法。

要求时间复杂度为 $\mathcal{O}(Q\log Q)$，空间线性。

----

先求出每个位置上的数最大可以填多少，设为 $b$，顺便判断是否有解：限制 $i$ 可能被满足当且仅当 $\max \limits_{j=l_i} ^{r_i} b_j = m_i$。

类似[[FJOI2017]矩阵填数 - 洛谷](https://www.luogu.com.cn/problem/P3813)这道题，$b_i$ 不同的两个位置的填法互不相关，可以依次求 $b_i=x$ 的所有位置的填法，再乘起来。

把位置、限制按 $b$ 分类，分别存起来，所有 $b_i=x$ 的位置把整个序列分成若干个左闭右开的区间。现在求 $b_i=x$ 的位置的填法数。注意 DP 时一个点不仅代表这一个点填什么，还代表了一个左闭右开的区间，设 $i$ 代表的区间长度为 $len_i$。

设计状态 $f_{i,j}$ 表示现在填到第 $i$ 个位置，上一个填 $x$ 的位置为 $j$，填法有 $f_{i,j}$ 种。

**一个观察：右端点在 $i$ 代表的区间中的所有限制，只要满足了左端点最靠右的一个，剩下的所有都满足了**。因此只需要考虑左端点最靠右的一个询问，设这个左端点为 $y$。则 $y$ 到 $i$ 中至少要有一个填 $x$。

如果 $i$ 不填 $x$，则 $y$ 到 $i-1$ 中至少有一个 $x$，所以 $\forall j: 0 \le j < y,\ f_{i,j}\leftarrow 0$，$\forall j : y \le j < i, \ f_{i,j} \leftarrow f_{i-1,j} \times (x-1)^{len_i}$。

如果填了 $x$，则 $f_{i,i}\leftarrow \sum \limits_{j=0} ^{i-1} f_{i-1,j} \times [x^{len_i} - (x-1)^{len_i}]$。

这里，位置编号是从 $1$ 开始的，$f_{i,0}$ 表示一个 $x$ 都没有。

答案即为 $\sum \limits_{i=0} ^{last} {f_{last,i} }$。

考虑优化，发现上面的转移可以转化为区间推平、区间乘、区间查询和单点修改，可以用线段树维护。

在实现时，区间查询的结果也可以用另外一个数组记录，看起来好看一点。

我的代码写得非常丑，好看的正解代码可以在洛谷 / AT 上找 qwq

AT 那道题和这题是一样的，只是没有多测。

## 其他题目

<https://www.luogu.com.cn/problem/P2605>

待补充

# 线段树合并优化 DP

在一些树上 DP 的题目中，需要把父亲和儿子的 DP 信息合并起来，而合并的过程可以转化为线段树合并操作。

## 例题 1. [[PKUWC2018] Minimax - 洛谷](https://www.luogu.com.cn/problem/P5298)

小 $C$ 有一棵 $n$ 个结点的有根树，根是 $1$ 号结点，且每个结点最多有两个子结点。

定义结点 $x$ 的权值为：

1.若 $x$ 没有子结点，那么它的权值会在输入里给出，**保证这类点中每个结点的权值互不相同**。

2.若 $x$ 有子结点，那么它的权值有 $p_x$ 的概率是它的子结点的权值的最大值，有 $1-p_x$ 的概率是它的子结点的权值的最小值。

现在小 $C$ 想知道，假设 $1$ 号结点的权值有 $m$ 种可能性，**权值第 $i$ 小**的可能性的权值是 $V_i$，它的概率为 $D_i(D_i>0)$，求：

$$
\sum_{i=1}^{m}i\cdot V_i\cdot D_i^2
$$

你需要输出答案对 $998244353$ 取模的值。

----

一眼树形 DP。设计状态 $f_{x,i}$ 表示节点 $x$ 取到权值 $i$ 的概率。

若 $x$ 为叶子节点，则 $f_{x,i} \leftarrow [val_x = i] val_x$；

若 $x$ 只有一个儿子，则 $f_{x,i} \leftarrow f_{son_x,i}$；

否则，设 $x$ 的两个儿子为 $lson$ 和 $rson$。

$$
f_{x,i} \leftarrow p_x\sum \limits_{j=1} ^{i-1} { f_{lson,j} f_{rson,i} } + (1-p_x)\sum \limits_{j=i+1} ^{V} f_{lson,j}f_{rson,i} +p_x\sum \limits_{j=1}^{i-1} {f_{lson,i}f_{rson,j} }+(1-p_x)\sum\limits_{j=i+1}^V { {f_{lson,i}f_{rson,j} }+{f_{lson,i}f_{rson,i} } }
$$

要怎么把线段树合并和这个东西结合起来呢？

$x$ 为叶子节点就是单点修改，只有一个儿子就是 `rt[x] = rt[sonx]`，主要问题是两个儿子的情况。

想一下线段树合并的过程：

```cpp
int merge(int x,int y,int l,int r)
{
    if (!x || !y) return x + y; // 其中一个为空就直接返回
    // tree[x] <- tree[y]
    if (l == r) return x; // 没法往下分就返回
    tree[x].ls = merge(tree[x].ls,tree[y].ls,l,mid);
    tree[x].rs = merge(tree[x].rs,tree[y].rs,mid + 1,r); // 递归
    return x;
}
```

假设现在 $f$ 数组已经成功地存在了两棵线段树里，要把 $f_{rson}$ 合并到 $f_{lson}$ 上作为 $f_x$。

现在合并到了区间 $[l,r]$。如果 $f_{rson}$ 在 $[l,r]$ 没有值，即 $y$ 为空，那么对于所有 $i \in [l,r]$，和它的值有关的 $f_{rson}$ 中的值都在这个区间外面，与 $i$ 本身无关，只与 $[l,r]$ 有关。具体来说，上面转移方程中，第一项、第二项和最后一项都是 $0$，第三项变为 $p_x\sum \limits_{j=1} ^{l-1} { f_{lson,i}f_{rson,j} }$，第四项变为 $(1-p_x)\sum \limits_{j=r+1}^V { f_{lson,i}f_{rson,j} }$。发现实际上就是给 $[l,r]$ 中的每个位置乘上 $p_x \sum \limits_{j=1}^{l-1} {f_{rson,j} }+(1-p_x)\sum \limits_{j=r+1}^V f_{rson,j}$，变成了区间乘。$x$ 为空时同理，因为 $x$ 和 $y$ 的关系是对等的。发现要维护一些前缀和和后缀和，在下分时顺便更新之后传下去就行了。

如果 $l=r$，直接根据转移方程暴力合并就行了。在这道题中，由于点的权值互不相同，所以不会有这种情况。

否则往下继续分就行了。注意下分前先下传 tag，合并完儿子之后再 pushup。

合并的代码：

```cpp
int merge(int x,int y,int l,int r,int xsum,int ysum,int P)
{
    if (!x && !y) return 0;
    if (!x || !y)
    {
        if (!x) swap(x,y),swap(xsum,ysum);
        tree[x].sum = 1ll * tree[x].sum * xsum % mod;
        tree[x].tg = 1ll * tree[x].tg * xsum % mod;
        return x;
    }
    pushdown(x),pushdown(y);
    int tmpx = tree[tree[x].ls].sum,tmpy = tree[tree[y].ls].sum;
    tree[x].ls = merge(tree[x].ls,tree[y].ls,l,mid,(xsum + 1ll * tree[tree[y].rs].sum * (1 - P + mod) % mod) % mod,(ysum + 1ll * tree[tree[x].rs].sum * (1 - P + mod) % mod) % mod,P);
    tree[x].rs = merge(tree[x].rs,tree[y].rs,mid + 1,r,(xsum + 1ll * tmpy * P % mod) % mod,(ysum + 1ll * tmpx * P % mod) % mod,P);
    pushup(x);
    return x;
}
```

剩下的都是动态开点线段树板子。

最后查询的时候，就是在根节点的线段树上单点查找。

## 例题 2. [[NOI2020] 命运 - 洛谷](https://www.luogu.com.cn/problem/P6773)

给定一棵树 $T = (V, E)$ 和点对集合 $\mathcal Q \subseteq V \times V$ ，满足对于所有 $(u, v) \in \mathcal Q$，都有 $u \neq v$，并且 $u$ 是 $v$ 在树 $T$ 上的祖先。其中 $V$ 和 $E$ 分别代表树 $T$ 的结点集和边集。求有多少个不同的函数 $f$ : $E \to \{0, 1\}$（将每条边 $e \in E$ 的 $f(e)$ 值置为 $0$ 或 $1$），满足对于任何 $(u, v) \in \mathcal Q$，都存在 $u$ 到 $v$ 路径上的一条边 $e$ 使得 $f(e) = 1$。由于答案可能非常大，你只需要输出结果对 $998,244,353$（一个素数）取模的结果。

$1 \le n \le 5 \times 10 ^5$，$1 \le m \le 5 \times 10^5$。

----

这道题和前面讲的 某位歌姬的故事 这道题很像，这道题就是把那道题的部分分做法搬到了树上。

这道题也有**一个观察：对于下端点相同的限制，上端点最深的满足，则这些限制都满足**。

设计状态 $f_{i,j}$ 表示在节点 $i$ 的子树中，还未被满足的限制中上端点最深的深度为 $j$，方案数为 $f_{i,j}$。根节点深度为 $1$。

转移方程：

$$
f_{x,i} \leftarrow \sum \limits_{j=0} ^{i - 1} { f_{x,i}f_{son,j} }+\sum\limits_{j=0} ^{i-1} { f_{x,j}f_{son,i} }+{ f_{x,i}f_{xon,i} }+f_{x,i}\sum \limits_{j=0}^{dep_x} {f_{son,j} }
$$

前面三项是 $edge(x,son)=0$，最后一项是填 $1$。

类似上一道题，这题也可以用线段树合并优化转移。但要注意，这道题里 $x$ 和 $son$ 的关系并不对等，要分别讨论左边为空和右边为空的情况。

合并代码：

```cpp
int merge(int x,int y,int l,int r,int pre,int sonpre,int sondep)
{
//    printf("%d %d %d %d %d %d %d\n",x,y,l,r,pre,sonpre,sondep);
    if (!x && !y) return 0;
    if (!x)
    {
        swap(x,y);
        tree[x].sum = 1ll * tree[x].sum * pre % mod;
//        printf("sum %d\n",tree[x].sum);
        tree[x].tg = 1ll * tree[x].tg * pre % mod;
        return x;
    }
    if (!y)
    {
        tree[x].sum = 1ll * tree[x].sum * (sondep + sonpre) % mod;
        tree[x].tg = 1ll * tree[x].tg * (sondep + sonpre) % mod;
        return x;
     }
     if (l == r)
     {
        tree[x].sum = (1ll * tree[x].sum * ((sondep + sonpre) % mod + tree[y].sum) % mod + 1ll * tree[y].sum * pre % mod) % mod;
//        printf("sum %d\n",tree[x].sum);
        return x;
    }
    pushdown(x),pushdown(y);
    int newpre = (pre + tree[tree[x].ls].sum) % mod;
    int newsonpre = (sonpre + tree[tree[y].ls].sum) % mod;
    tree[x].ls = merge(tree[x].ls,tree[y].ls,l,mid,pre,sonpre,sondep);
    tree[x].rs = merge(tree[x].rs,tree[y].rs,mid + 1,r,newpre,newsonpre,sondep);
    pushup(x);
    return x;
}
```

# 总结

遇到整个区间转移的问题，可以往线段树优化上考虑。（？）
