---
title: AC 自动机 学习笔记
date: 2021-12-07 19:24:43
tags:
    - 字符串
    - 笔记
categories: OI
mathjax: true
description: AC 自动机 学习笔记
---

# 前置知识

- Trie 树
- KMP 的思想（也许不用？）

# AC自动机是什么？

~~可以帮你自动A题的作弊玩意~~

> AC 自动机是 以 Trie 的结构为基础，结合 KMP 的思想 建立的。
> 简单来说，建立一个 AC 自动机有两个步骤：
> 1.基础的 Trie 结构：将所有的模式串构成一棵 Trie。
> 2.KMP 的思想：对 Trie 树上所有的结点构造失配指针。
> 然后就可以利用它进行多模式匹配了。
> ——OI-Wiki

# 模板题

<https://www.luogu.com.cn/problem/P3808>
给定 $n$ 个模式串 $s_{1,\dots ,n}$ 和一个文本串 $t$，求有多少个不同的模式串在文本串里出现过。
两个模式串不同当且仅当他们**编号**不同。

$1 \leq n \leq 1e6$，$1 \leq |t| \leq 1e6$，$1 \leq \sum \limits_{i=1} ^n |s_i| \leq 1e6$。

# 算法流程

对于上面这道题，显然我们可以对于每个模式串都判断一次是否在文本串里，时间复杂度 $O(n · |t|)$。很明显过不了。

这时，我们想到：KMP 算法中，我们利用了**失配指针**减少了不必要的比较，提高了算法效率！这种想法是否也可以用在这一题里呢？

显然是可以的。

考虑两个模式串 ```abcd``` 和 ```bce```，文本串为 ```abce```。

对于第一个模式串，当我们判断到文本串 ```e``` 这个字符时，发现不对了。如果按照上面的思路，我们需要从头再枚举，判断第二个模式串是否出现。但是，我们观察到，```bce``` 的前两个字符在判断一个模式串时就已经被比较过了，产生了不必要的比较。因此，我们可以用一个失配指针，从第一个模式串的 ```c``` 指向第二个模式串的 ```c```。

存储模式串，我们使用 Trie 树。

通过上面的这个例子，我们可以发现：当一个模式串的某一前缀为另一模式串的子串时，可以用上失配指针。

具体地说，若 $S_i(1,k) = S_j(x,x+k-1)$，就产生一个从 $S_j[x+k-1]$ 到 $S_i[k]$ 的失配指针，因为当 $S_j(1,x+k-1)$ 在文本串中时，$S_j(x,x+k-1)=S_i(1,k)$ 也一定在文本串中，我们无需比较 $S_i$ 的前  $k$ 位。

通过一段分析，我们已经大致得出了算法的思路：存模式串，建失配指针，查找。

接下来就只剩细节问题了。

## 失配指针的建立

我们使用 BFS 建立失配指针。

来看看这样的一个 Trie 树。

（Trie 树的所有节点都代表了一个字符串，所有边都代表了一个字符）

（图待补充）

很明显，对于第一层的所有节点，失配指针只能指向根节点。

对于第二层及以下的层：

设当前正在考虑的节点为 $i$，它代表的字符串为 $tree[i]$，其父亲为 $fa[i]$，$x$ 的失配指针指向的节点为 $fail[x]$，连接 $x$ 和 $y$ 的边代表的字符为 $c(x,y)$。

根据失配指针的定义，$tree[fa[i]](x_0,x_0+k-1)=tree[fail[fa[i]]](1,k)$。

若 $i$ 存在：
若 $fail[fa[i]]$ 的一个儿子 $r$ 满足 $c(fa[i],i)=c(fail[fa[i]],r)$，则 $tree[i](x_0,x_0+k)=tree[r](1,k+1)$，即 $i$ 的失配指针应指向 $r$。否则 $i$ 的失配指针指向根节点。同时将 $i$ 入队。

若 $i$ 这个节点不存在，则给 $fa[i]$ 和 $fail[fa[i]]$ 的满足 $c(fa[i],i)=c(fail[fa[i]],r)$ 的儿子 $r$ 连一条边。

这样，经过一次 BFS，失配指针就建立好了。

## 查找答案

和暴力的思路差不多，在 Trie 树中，一个一个字符查找文本串，如果到了一个模式串的末尾就更新答案，如果匹配不上就跳失配指针。

# 代码

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<queue>
using namespace std;

int n;
char s[1000010];
int tr[1000010][30],tg[1000010],fail[1000010];
int cnt;

void add(char s[])
{
    int nw = 0,len = strlen(s);
    for (int i = 0;i < len;i ++)
    {
        if (!tr[nw][s[i] - 'a'])
            tr[nw][s[i] - 'a'] = ++ cnt;
        nw = tr[nw][s[i] - 'a'];
    }
    tg[nw] ++;
}

queue <int> q;
void build()
{
    for (int i = 0;i < 26;i ++)
        if (tr[0][i]) q.push(tr[0][i]);
    while (q.size())
    {
        int nw = q.front();
        q.pop();
        for (int i = 0;i < 26;i ++)
            if (tr[nw][i])
                fail[tr[nw][i]] = tr[fail[nw]][i],q.push(tr[nw][i]);
            else tr[nw][i] = tr[fail[nw]][i];
    }
}

void query(char s[])
{
    int nw = 0,ans = 0,len = strlen(s);
    for (int i = 0;i < len;i ++)
    {
        nw = tr[nw][s[i] - 'a'];
        for (int j = nw;j && tg[j] != -1;j = fail[j])
        {
            ans += tg[j];
            tg[j] = -1;
        }
    }
    printf("%d\n",ans);
}

int main()
{
    scanf("%d",&n);
    for (int i = 1;i <= n;i ++)
    {
        scanf("%s",s);
        add(s);
    }
    build();
    scanf("%s",s);
    query(s);
}
```

# 例题

<https://www.luogu.com.cn/problem/P3808>（模板）<https://www.luogu.com.cn/problem/P3796>（加强版）
（待补充）
