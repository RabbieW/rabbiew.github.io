---
title: 树上启发式合并（dsu on tree）学习笔记
date: 2024-02-26 11:43:53
tags:
  - 树
  - 笔记
categories: OI
mathjax: true
description: 树上启发式合并的自学&上课笔记
---

# 树上启发式合并是什么？

树上启发式合并（dsu on tree）通过继承重儿子的信息，可以将某些合并更新答案次数为 $O(n^2)$ 的问题优化至 $O(n \log n)$。

# 算法过程

在普通 dfs 的基础上，继承重儿子传上来的信息。

先 dfs 轻儿子并把记录的信息清空，再 dfs 重儿子且不清空信息（继承）。随后暴力搜索轻子树内的节点更新信息。

这样做可以保证每次传上来的只有重儿子的信息，且算完当前子树的答案后，目前保留的信息只有这棵子树的。

代码：

```cpp
void dfss(int x)
{
    for (int i = head[x];i;i = nxt[i])
    {
        int u = to[i];
        if (u == son[x]) continue;
        dfss(u);//递归处理轻儿子
        clr(u);//清空
    }
    if (son[x]) dfss(son[x]);
    ...//加入当前节点信息
    for (int i = head[x];i;i = nxt[i])
    {
        int u = to[i];
        if (u == son[x]) continue;
        upd(u,x);//更新答案
        add(u);//更新信息
    }
}
```

# 时间复杂度

对每一个节点，如果它是父亲的轻儿子，它所在子树就会在 dfs 父亲时被暴力更新一次。也就是说，一个点被暴力更新的次数即为它到根经过的轻边数，是 $O(\log n)$ 的。所以总的更新次数是 $O(n \log n)$ 的。

## 注意

为了保证时间复杂度正确，继承重儿子信息这一步要保证更新次数为 $O(1)$的，所以一般存信息时，要存不会因为当前节点变化而变化很大的信息。

另外，清空信息的复杂度也要注意不要爆炸。一般直接 dfs 子树清空即可。

# 题目

[这篇文章](https://codeforces.com/blog/entry/44351)的最后有一堆题目。

[CF1923E](https://codeforces.com/contest/1923/problem/E)

[CF778C](https://codeforces.com/problemset/problem/778/C)，甚至不用传信息

[[Cnoi2019] 雪松果树](https://www.luogu.com.cn/problem/P5384)，虽然我写的 $O(n)$ 做法跑得比 $O(n \log n)$ 还慢。。。

p.s: 一个很像但是似乎不一样的东西：带撤销的完全背包的空间优化 [[Cnoi2019] 青染之心](https://www.luogu.com.cn/problem/P5391)，通过将问题转化为类似树上的问题，重链剖分后同一条链用一个数组、深度相同的用同一个数组，把空间从 $O(nV)$ 优化到 $O(V\log n)$。这里是从上到下的信息传递。